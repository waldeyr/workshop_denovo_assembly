# Workshop de novo Assembly and annotation of bacterial drug resistance genes

The objective of this workflow is to assembly the genome of pathogen [Enterobacter cloacae subsp. cloacae NCTC 9394](https://www.ncbi.nlm.nih.gov/biosample/SAMEA3181516) and find drug-resistent genes

Design: Illumina sequencing of library 2371033, constructed from sample accession ERS024397 for study accession ERP000539. 
Library: 2371033
Instrument: Illumina HiSeq 2000
Strategy: WGS
Source: GENOMIC
Selection: RANDOM
Layout: PAIRED
Construction protocol: Standard
Runs: 1 run, 6.5M spots, 1.3G bases, 885.7Mb

## Environment prepararion (Based on a Ubuntu Docker Image with root user)

`apt update -y && apt upgrade -y`

`apt install -y wget python3-pip python3-dev python-dev zip unzip default-jdk curl nano git gcc gcc zlib1g-dev zlib1g build-essential r-base pkg-config libfreetype6-dev libpng-dev python-matplotlib ncbi-blast+`

`pip3 install --upgrade cutadapt biopython`

`curl -L http://cpanmin.us | perl - --force Time::HiRes`

### SRA Toolkit 2.9.6-1

`wget https://ftp-trace.ncbi.nlm.nih.gov/sra/sdk/current/sratoolkit.current-ubuntu64.tar.gz && tar -xzf sratoolkit.current-ubuntu64.tar.gz && rm sratoolkit.current-ubuntu64.tar.gz`

`ln -sfv /home/sratoolkit.2.9.6-1-ubuntu64/bin/* /usr/local/bin/`

### Trimmomatic-0.39

`wget http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/Trimmomatic-0.39.zip && unzip Trimmomatic-0.39.zip && rm Trimmomatic-0.39.zip`

### SPAdes-3.13

`wget http://cab.spbu.ru/files/release3.13.0/SPAdes-3.13.0-Linux.tar.gz && tar -xzf SPAdes-3.13.0-Linux.tar.gz && rm SPAdes-3.13.0-Linux.tar.gz`

### Quast-5.0.2

`wget https://sourceforge.net/projects/quast/files/quast-5.0.2.tar.gz && tar -xzf quast-5.0.2.tar.gz && rm quast-5.0.2.tar.gz`

`quast-5.0.2/install_full.sh && rm -rf quast_test_output`

### Glimmer-3.02

`wget http://ccb.jhu.edu/software/glimmer/glimmer302b.tar.gz && tar -xzf glimmer302b.tar.gz && rm glimmer302b.tar.gz && cd glimmer3.02/src/ && make && cd /home/bioinfo`


## Workflow execution

Doownload the docker image with the environment

`docker pull waldeyr/bioinfo:drug_resistance`

Run a docker container with the image

`docker run --rm -it waldeyr/bioinfo:drug_resistance`

### Downloading Sequences

`cd /home`

* Illumina sequencing reads library 

`fastq-dump -v --split-files ERR037801`

* Set of drug resistence genes (MEGARes, doi:10.1093/nar/gkw1009)

`wget https://megares.meglab.org/download/megares_v1.01/megares_database_v1.01.fasta`

### Filtering

`java -jar /home/bioinfo/Trimmomatic-0.39/trimmomatic-0.39.jar PE -phred33 ERR037801_1.fastq ERR037801_2.fastq ERR037801_1_FILTERED.fastq ERR037801_1_UNPAIRED.fastq ERR037801_2_FILTERED.fastq ERR037801_2_UNPAIRED.fastq ILLUMINACLIP:/home/bioinfo/Trimmomatic-0.39/adapters/TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36 -validatePairs`

### Assembly

`python3 /home/bioinfo/SPAdes-3.13.0-Linux/bin/spades.py --pe1-1 ERR037801_1_FILTERED.fastq --pe1-2 ERR037801_2_FILTERED.fastq -t 4 --careful --cov-cutoff auto -o ERR037801_assembly`

### Assembly stats

`/home/bioinfo/quast-5.0.2/quast.py ERR037801_assembly/contigs.fasta`

### Resistance genes prediction

`/home/bioinfo/glimmer3.02/bin/build-icm MEGARes < megares_database_v1.01.fasta`

`/home/bioinfo/glimmer3.02/bin/glimmer3 ERR037801_assembly/contigs.fasta MEGARes drug_resistence_genes`

`/home/bioinfo/glimmer3.02/bin/extract ERR037801_assembly/contigs.fasta drug_resistence_genes.predict > putative_drug_resistence_genes.fasta`

### Gene annotation

`makeblastdb -dbtype nucl -in megares_database_v1.01.fasta -out MEGARes`

`blastn -query putative_drug_resistence_genes.fasta -evalue 10e-5 -db MEGARes -out putative_drug_resistence_genes_annotated.fasta`


