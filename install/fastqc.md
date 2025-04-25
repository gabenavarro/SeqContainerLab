## FastQC-RC

A fast quality control tool for FASTQ files written in rust.

* [Home](https://fastqc-rs.github.io)
* [Development](https://github.com/fastqc-rs/fastqc-rs)

### Prequisit

1. Docker
2. Git
3. Cloud SDK
    - [GCP](./gcp_setup.md)
    - [AWS](./aws_setup.md)

### Install 

#### Locally

Pull repository

```bash
git clone https://github.com/gabenavarro/SeqContainerLab.git
cd SeqContainerLab
```

Build docker image

```bash
docker build \
-f ./assets/build/Dockerfile.fastqc.illumina \
-t fastqc-rs:0.3.4 .
```

#### Cloud

##### Google
```bash
# Tag docker image for GCP artifact registry
docker tag \
fastqc-rs:0.3.4 \
<gpc.artifact.registry>/fastqc-rs:0.3.4
# Push tagged image to GCP artifact registry
docker push \
<gpc.artifact.registry>/fastqc-rs:0.3.4
```


### Run

#### Locally

1. Download test dataset. 
    * [Genome sequence of a Bacillus subtilis ALBA01 strain](https://www.ebi.ac.uk/ena/browser/view/SRR3317165)

    ```bash
    # Make directory, that is gitignored
    mkdir data
    # Download FastQ files
    wget -nc -P ./data ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR331/005/SRR3317165/SRR3317165_2.fastq.gz
    wget -nc -P ./data ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR331/005/SRR3317165/SRR3317165_1.fastq.gz
    ```
    * To find more publicly available fastq file, search for `Read` in the [European Nucleotide Archive (ENA)](https://www.ebi.ac.uk/ena/browser/home). 

2. Run QC Analysis via docker run

```bash
docker run --rm -it \
  -v "$(pwd):/app" \
  fastqc-rs:0.3.4 \
  --user 1000:1000 \
  bash -c \
  "fqc -q /app/data/SRR3317165_1.fastq.gz > /app/data/SRR3317165_1.html"
```