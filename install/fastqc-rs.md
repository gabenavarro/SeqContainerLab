# FastQC-RS: FASTQ Quality Control in Rust

**FastQC-RS** is a high-performance, lightweight tool for quality control analysis of FASTQ sequencing files, written in Rust.

- [Home Page](https://fastqc-rs.github.io)
- [GitHub Repository](https://github.com/fastqc-rs/fastqc-rs)

---

## 📚 Prerequisites

Before you begin, ensure you have the following installed:

- [Docker](https://docs.docker.com/get-docker/)
- [Git](https://git-scm.com/)
- (Optional for Cloud Deployment)
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
  -f ./assets/build/Dockerfile.fastqcrs \
  -t fastqc-rs:0.3.4 .
```

This will build a clean, lightweight container with **FastQC-RS v0.3.4** installed.

---

## ☁️ Cloud Installation (Google Cloud Platform)

### 1. Build the Image Locally

Follow the same local build instructions above to create the Docker image.

### 2. Tag the Docker Image

Replace `<gcp.artifact.registry>` with your Google Artifact Registry path:

```bash
docker tag fastqc-rs:0.3.4 <gcp.artifact.registry>/fastqc-rs:0.3.4
```

Example:
```bash
docker tag fastqc-rs:0.3.4 us-central1-docker.pkg.dev/my-project-id/my-repo/fastqc-rs:0.3.4
```

### 3. Push the Image to GCP

```bash
docker push <gcp.artifact.registry>/fastqc-rs:0.3.4
```

> **Note:** You must be authenticated to GCP and have Artifact Registry permissions configured.

---

# 🚀 Running FastQC-RS

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

### 2. Run FastQC-RS on a FASTQ File

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  --user 1000:1000 \
  fastqc-rs:0.3.4 \
  bash -c \
  "fqc -q /app/data/SRR3317165_1.fastq.gz \
  > /app/data/SRR3317165_1.html"
```

### ✅ Output

- The quality control report (`.html`) will be saved inside the `data/` directory.

---

## 📋 Notes and Tips

- You can easily batch-process multiple FASTQ files by looping over input files.

* **Docker Options Explained:**
- `-v "$(pwd):/app"` mounts your local directory into the Docker container.
- `--user 1000:1000` ensures that output files have correct permissions for local editing.
- `--rm` cleans up the container after execution.
> For more options, refer to the [Docker documentation](https://docs.docker.com/engine/reference/run/).

* **FastQC-RS Options:**
- `-q` specifies the input FASTQ file.
- `>` redirects the output to a specified file.
> For more options, refer to the [FastQC-RS documentation](https://fastqc-rs.github.io/).



---

# 🔮 Future Enhancements (Optional)

- Add multi-file batch processing.
- Cloud-native pipeline integrations (Vertex AI Pipelines, AWS Batch).
- Automated report summarization (HTML dashboards).
