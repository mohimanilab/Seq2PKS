# Seq2PKS

Discovering modular type I Cis-AT polyketide natural products by integrating computational mass spectrometry and genome mining

## Introduction

Type 1 Polyketides are a major class of natural products that are used as antiviral, antibiotic, antifungal, antiparasitic, immunosuppressant, and antitumor drugs. A recent analysis of hundreds of thousands of public microbial genomes has resulted in the discovery of over sixty thousand Type 1 Polyketide gene clusters. Despite these efforts, the molecular products for only 80 of these gene clusters have been characterized, and the product for the majority of these gene clusters remains unknown. The existing technology for characterizing novel polyketides is based on bioactivity-guided purification, an expensive and time-consuming effort that is limited to highly abundant molecular products. 

To overcome this challenge and identify the molecular product of polyketide gene clusters in high-throughput, we present Seq2PKS, a machine learning algorithm for predicting the chemical structure of the molecular product of Type 1 polyketide gene clusters. To enhance accuracy, Seq2PKS predicts hundreds or thousands of putative structures for each gene cluster, and then the correct structure is identified among these predictions using a mass spectral database search. 

![seq2pks_overview](images\seq2pks_overview.png)

## Seq2PKS pipeline

The Seq2PKS pipeline contains five major steps:

### Detect polyketide Biosynthetic gene clusters (BGCs), Modules and Domains

This step use antismash to detect polyketide Biosynthetic gene clusters (BGCs), Modules and Domains. The parsed result from antismash are stored in the folder **parsed_result**.

### Identify AT domain specificity 

In this step, a pre-trained machine learning model is used to predict the AT domain specificity for all AT domain in each BGC. The result for this step are being stored in the folder **specificity_result**.

### Predict structure order

In this step, the assemble order for monomers being recruited by each gene in the BGC is being predicted. The result for this step are being sotred in the folder **rank.**

### Construct backbone

In this step, The AT domain specificity are combined with other domain information to obtain the monomer correspond to each module. These monomers are then being connected to form the backbone following the order predicted in the previous step. The result for this step are being sotred in the folder **backbone**.

### apply post-modification and score against input spectrum file. 

In this step, the genes that responsible for post-modification are being detected using HMM search. Then the corresponding modifications are being applied to the backbone using core2pks. The candidate compounds are scored against spectrum using dereplicator+ or moldiscovery. The result for this step are being sotred in the folder **compound**.

## Installing Seq2PKS

### Pre-built Docker image(Recommended)

```
wget https://github.com/mohimanilab/Seq2PKS/main/docker/seq2pks.tar
sudo docker load -i seq2pks.tar
```

### Manually install on Unix-based system(Not recommended)

#### Installing Other Dependencies

If not present you will also need to install:

- [emboss](https://emboss.sourceforge.net/)
- [ncbi-blast+](https://blast.ncbi.nlm.nih.gov/doc/blast-help/downloadblastdata.html#downloadblastdata)

For example, on Ubuntu,

```
sudo apt-get install emboss ncbi-blast+
```

- [singularity](https://github.com/sylabs/singularity/releases)

Follow the [instructions](https://docs.sylabs.io/guides/4.0/user-guide/quick_start.html) to install it.

#### Local environment

Below packages are needed along with python.

* `git`
* `pandas`
* `numpy`
* `rdkit`
* `antismash`
* `scikit-learn`
* `biopython`

For example, using `conda`,

```
conda create -n "Seq2PKS" pandas numpy rdkit antismash scikit-learn biopython -c conda-forge -c bioconda
conda activate Seq2PKS
git clone https://github.com/mohimanilab/Seq2PKS.git
```

Or using `mamba`, if the process is being killed when installing `antismash` package.

```
conda install mamba
mamba create -n "Seq2PKS" pandas numpy rdkit antismash scikit-learn biopython -c conda-forge -c bioconda
conda activate Seq2PKS
git clone https://github.com/mohimanilab/Seq2PKS.git
```

## Running Seq2PKS

Seq2PKS currently can take either Ncbi_id or fasta file as input. Here are the input parameters for the software.

* `ncbi_id` ncbi_id for the input sequence
* `sequence_file` path to the sequence file
* `sequence_id` name for the sequence file
* `output_folder` output folder to store the result
* `spectrum_path` path to the spectrum file
* `pattern` algorithm for spectrum matching, can take "moldiscovery" or "dereplicator+" as input
* `n_jobs` Number of parallel jobs during computation
* `smile_compound`: Expected SMILE string for the compound if known

### Docker(Recommended)

Create your working directory,

```
mkdir Seq2PKS
cd Seq2PKS
```

Interacting with the container,

```
sudo docker run -it -v $(pwd):/usr/src/app/mnt --privileged --entrypoint /bin/bash seq2pks
```

Here is an example of running with a FASTA file as input:

```
cp /path/to/source/{sample.fasta,sample_spectra.mzML} .
python main.py --sequence_file sample.fasta --sequence_id sample --pattern dereplicator_plus --spectrum_path sample_spectra.mzML --output_folder ./mnt/result
```

We have the sample run result included in the "test_result" folder for testing purposes. You should be able to generate the exact same result in the folder by following the below command,

```
python main.py --ncbi_id DQ149987.1 --pattern dereplicator_plus --spectrum_path sample_spectram.mzML --smile_compound 'CC=1CC(C)C(O)C(CC)C(O)C(C)C=C(C)C=C(OC)C(=O)OC(C(C(C(C3(O)CC(C(C(C=CC)O3)C)OC2OC(C(OC(=O)N)C(O)C2)C)C)O)C)C(C=CC=1)OC' --output_folder test_result
cp -r test_result mnt
```

The identified compounds are stored in the file "test_result/DQ149987/compound/backbone2pks_result/database.csv". 
The generated compounds are scored against the input spectrum file using dereplicator+, the obtained match is stored in "test_result/DQ149987/compound/backbone2pks_result/psms.tsv".

### Unix-based system(Not recommended)

Here is an example of running with a FASTA file as input:

```
cd Seq2PKS
cp /path/to/source/{sample.fasta,sample_spectra.mzML} .
python main.py --sequence_file sample.fasta --sequence_id sample --pattern dereplicator_plus --spectrum_path sample_spectra.mzML --output_folder result
```

We have the sample run result included in the "test_result" folder for testing purposes. You should be able to generate the exact same result in the folder by following the below command,

```
python main.py --ncbi_id DQ149987.1 --pattern dereplicator_plus --spectrum_path sample_spectram.mzML --smile_compound 'CC=1CC(C)C(O)C(CC)C(O)C(C)C=C(C)C=C(OC)C(=O)OC(C(C(C(C3(O)CC(C(C(C=CC)O3)C)OC2OC(C(OC(=O)N)C(O)C2)C)C)O)C)C(C=CC=1)OC' --output_folder test_result
```

The identified compounds are stored in the file "test_result/DQ149987/compound/backbone2pks_result/database.csv". 
The generated compounds are scored against the input spectrum file using dereplicator+, the obtained match is stored in "test_result/DQ149987/compound/backbone2pks_result/psms.tsv".

## Seq2PKS web server

The web server for Seq2PKS will be available soon at npanalysis.org.
