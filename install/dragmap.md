# DragMap-GATK Installation and Usage Guide

**DragMap** is Illumina's open-source implementation of the DRAGEN mapper/aligner — a powerful tool for mapping sequencing reads to a reference genome. It is designed for high-throughput sequencing data and integrates seamlessly with GATK.  
This guide provides step-by-step instructions for installing and running **DragMap** alongside **GATK** using Docker.

- [🏠 DragMap GitHub](https://github.com/Illumina/DRAGMAP)  
- [🧬 GATK GitHub](https://github.com/broadinstitute/gatk)  
- [📖 Documentation Overview](https://gatk.broadinstitute.org/hc/en-us/articles/4410953761563-Introducing-DRAGMAP-the-new-genome-mapper-in-DRAGEN-GATK)

---

## 📚 Prerequisites

Make sure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Git](https://git-scm.com/)
- *(Optional for Cloud Deployment)*:
  - [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)
  - [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

---

# 🛠️ Installation Guide

## 🖥️ Local Installation (Development or Personal Use)

### 1. Clone the Repository

```bash
git clone https://github.com/gabenavarro/SeqContainerLab.git
cd SeqContainerLab
```

### 2. Build the Docker Image

```bash
docker build \
  -f ./assets/build/Dockerfile.dragmap \
  -t dragmap:1.3.0 .
```

> **Note:** This builds a Docker image based on a Gambalab-maintained base, bundling **GATK 4**, **DragMap**, and **Samtools**. It is **not ultra-lightweight** but is convenient for local development and prototyping.

---

## ☁️ Cloud Installation (Google Cloud Platform)

1. **Build the Image Locally**

Follow the local build instructions above.

2. **Tag the Docker Image**

Replace `<gcp.artifact.registry>` with your Artifact Registry path:

```bash
docker tag dragmap:1.3.0 <gcp.artifact.registry>/dragmap:1.3.0
```

Example:

```bash
docker tag dragmap:1.3.0 us-central1-docker.pkg.dev/my-project-id/my-repo/dragmap:1.3.0
```

3. **Push the Image to GCP**

```bash
docker push <gcp.artifact.registry>/dragmap:1.3.0
```

> **Note:** Make sure you are authenticated to GCP and have Artifact Registry permissions enabled.

---

# 🚀 Running DragMap

## 🖥️ Local Execution

### 1. Prepare the Reference Genome

**Download Reference Genome:**

```bash
mkdir -p data

wget -nc -P ./data ftp://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/009/045/GCF_000009045.1_ASM904v1/GCF_000009045.1_ASM904v1_genomic.fna.gz
```

**Unzip Reference Genome:**

```bash
gunzip ./data/GCF_000009045.1_ASM904v1_genomic.fna.gz
```

**Create FASTA Dictionary (GATK):**

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  dragmap:1.3.0 \
  bash -c '
  gatk CreateSequenceDictionary \
    -R "/app/data/GCF_000009045.1_ASM904v1_genomic.fna"'
```

**Index Reference (Samtools):**

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  dragmap:1.3.0 \
  bash -c '
  samtools faidx "/app/data/GCF_000009045.1_ASM904v1_genomic.fna"'
```

**Compose STR Table (GATK):**

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  dragmap:1.3.0 \
  bash -c '
  gatk ComposeSTRTableFile \
    -R "/app/data/GCF_000009045.1_ASM904v1_genomic.fna" \
    -O "/app/data/GCF_000009045.1_ASM904v1_genomic.fna.strtable"'
```

---

### 2. Prepare FASTQ Files

Follow the [FastP Setup Guide](install/fastp.md) for instructions on how to obtain and preprocess FASTQ files.  
We will use the same *Bacillus subtilis* dataset as shown there.

---

### 3. Running DragMap: Mapping Reads

#### Build Hash Table

Before alignment, DragMap requires a hash table built from the reference:

```bash
mkdir -p data/hash_table

docker run --rm -it \
  -v "$(pwd):/app" \
  dragmap:1.3.0 \
  bash -c '
  dragen-os \
    --build-hash-table true \
    --ht-reference "/app/data/GCF_000009045.1_ASM904v1_genomic.fna" \
    --output-directory "/app/data/hash_table/" \
    --ht-write-hash-bin 1 \
    --num-threads 16'
```

#### Map Reads and Create BAM

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  dragmap:1.3.0 \
  bash -c '
  dragen-os \
    -r "/app/data/hash_table/" \
    -1 "/app/data/SRR3317165_1.trim.fastq.gz" \
    -2 "/app/data/SRR3317165_2.trim.fastq.gz" \
    --num-threads 16 \
  | samtools view \
    --threads 16 \
    -bh -o "/app/data/SRR3317165.bam"'
```

> **Note:** This pipes DragMap output directly into `samtools view` to generate a compressed BAM file on-the-fly.

---

### ✅ Output Summary

| Output File | Description |
| ----------- | ----------- |
| `GCF_000009045.1_ASM904v1_genomic.fna` | Reference genome FASTA |
| `GCF_000009045.1_ASM904v1_genomic.fna.fai` | FASTA index |
| `GCF_000009045.1_ASM904v1_genomic.dict` | Sequence dictionary |
| `GCF_000009045.1_ASM904v1_genomic.fna.strtable` | STR table for reference |
| `data/hash_table/` | DragMap hash table |
| `SRR3317165.bam` | Aligned sequencing reads (BAM) |

---

# 📋 Notes and Tips

**Docker Tips:**
- `-v "$(pwd):/app"` mounts your local directory inside the container.
- `--rm` automatically deletes the container after completion.

**DragMap Key Parameters:**
- `--build-hash-table`: Flag to build reference hash table.
- `--ht-reference`: Path to reference FASTA.
- `--output-directory`: Output directory for hash table files.
- `-r`: Directory of hash table for alignment.
- `-1` and `-2`: Input FASTQ files (paired-end reads).
- `--num-threads`: Number of threads for performance optimization.

**Samtools Key Parameters:**
- `samtools view --threads 16 -bh`: Convert SAM to compressed BAM format using multiple CPU threads.

---

# 🔮 Future Enhancements (Optional)

- Automate multi-sample processing pipelines.
- Cloud-native integration with Vertex AI Pipelines or AWS Batch.
- Add GATK post-processing steps (e.g., MarkDuplicates, BaseRecalibrator).
- Dashboard summarization of alignment metrics.
