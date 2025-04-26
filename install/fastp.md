# FastP: FASTQ Preprocessing and Quality Control

**FastP** is an ultrafast all-in-one FASTQ preprocessing tool for **short read sequencing data**. It offers both quality control and cleaning in a single step. Actively maintained and optimized for performance.

- [🏠 Home Page](https://github.com/OpenGene/fastp/tree/v0.24.1)
- [📖 Documentation](https://github.com/OpenGene/fastp/blob/v0.24.1/README.md)

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
  -f ./assets/build/Dockerfile.fastp \
  -t fastp:0.24.1 .
```

> This builds a lightweight container with **fastp v0.24.1** installed.

---

## ☁️ Cloud Installation (Google Cloud Platform)

### 1. Build the Image Locally

Follow the same instructions above for building the Docker image.

### 2. Tag the Docker Image

Replace `<gcp.artifact.registry>` with your Google Artifact Registry path:

```bash
docker tag fastp:0.24.1 <gcp.artifact.registry>/fastp:0.24.1
```

Example:
```bash
docker tag fastp:0.24.1 us-central1-docker.pkg.dev/my-project-id/my-repo/fastp:0.24.1
```

### 3. Push the Image to GCP

```bash
docker push <gcp.artifact.registry>/fastp:0.24.1
```

> **Note:** Ensure you are authenticated to GCP and have Artifact Registry permissions configured.

---

# 🚀 Running FastP

## 🖥️ Local Execution

### 1. Download Example FASTQ Data

Example dataset: [*Bacillus subtilis* ALBA01 genome sequence](https://www.ebi.ac.uk/ena/browser/view/SRR3317165).

```bash
# Create a data directory (already gitignored)
mkdir -p data

# Download paired-end FASTQ files
wget -nc -P ./data ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR331/005/SRR3317165/SRR3317165_1.fastq.gz
wget -nc -P ./data ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR331/005/SRR3317165/SRR3317165_2.fastq.gz
```

> You can explore additional datasets at the [European Nucleotide Archive (ENA)](https://www.ebi.ac.uk/ena/browser/home).

---

### 2. Run FastP on FASTQ Files

**Paired-End FASTQ Processing:**

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  --user 1000:1000 \
  fastp:0.24.1 \
  bash -c '
    fastp \
      --in1 "/app/data/SRR3317165_1.fastq.gz" \
      --in2 "/app/data/SRR3317165_2.fastq.gz" \
      --out1 "/app/data/SRR3317165_1.trim.fastq.gz" \
      --out2 "/app/data/SRR3317165_2.trim.fastq.gz" \
      --unpaired1 "/app/data/SRR3317165_1.trim_up.fastq.gz" \
      --unpaired2 "/app/data/SRR3317165_2.trim_up.fastq.gz" \
      --qualified_quality_phred 20 \
      --detect_adapter_for_pe \
      --length_required 50 \
      --correction \
      --low_complexity_filter \
      --complexity_threshold 30 \
      --html /app/data/fastp.html \
      --json /app/data/fastp.json \
      --thread 16'
```

---

### ✅ Output

FastP will generate:

- `SRR3317165_1.trim.fastq.gz` — Trimmed R1 reads
- `SRR3317165_2.trim.fastq.gz` — Trimmed R2 reads
- `SRR3317165_1.trim_up.fastq.gz` — Unpaired reads from R1
- `SRR3317165_2.trim_up.fastq.gz` — Unpaired reads from R2
- `fastp.html` — Interactive HTML report
- `fastp.json` — JSON-formatted QC report

---

## 📋 Notes and Tips

**Docker Options:**
- `-v "$(pwd):/app"` mounts your current working directory into the container.
- `--user 1000:1000` ensures output files are owned by your user (not root).
- `--rm` automatically removes the container after execution.
> For more options, refer to the [Docker documentation](https://docs.docker.com/engine/reference/run/).

**FastP Key Parameters:**
- `--in1` / `--in2`: Input paired-end FASTQ files.
- `--out1` / `--out2`: Trimmed output paired-end FASTQ files.
- `--unpaired1` / `--unpaired2`: Output unpaired reads.
- `--qualified_quality_phred`: Minimum base quality threshold.
- `--detect_adapter_for_pe`: Automatically detect adapters.
- `--length_required`: Minimum read length to keep after trimming.
- `--correction`: Enable correction for overlapping paired-end reads.
- `--low_complexity_filter`: Remove low-complexity reads.
- `--complexity_threshold`: Minimum sequence complexity threshold.
- `--html` / `--json`: Output summary reports in HTML and JSON formats.
- `--thread`: Number of CPU threads to use.
> For a full list of options, refer to the [FastP documentation](https://github.com/OpenGene/fastp/blob/master/README.md)

---

# 🔮 Future Enhancements (Optional)

- Batch processing multiple FASTQ samples in one command.
- Cloud-native pipeline integrations (e.g., Vertex AI Pipelines, AWS Batch).
- Auto-generation of multi-sample quality summary dashboards.
