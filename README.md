# Seq2PKS

Discovering modular type I Cis-AT polyketide natural products by integrating computational mass spectrometry and genome mining

## Introduction

Type 1 Polyketides are a major class of natural products that are used as antiviral, antibiotic, antifungal, antiparasitic, immunosuppressant, and antitumor drugs. A recent analysis of hundreds of thousands of public microbial genomes has resulted in the discovery of over sixty thousand Type 1 Polyketide gene clusters. Despite these efforts, the molecular products for only 80 of these gene clusters have been characterized, and the product for the majority of these gene clusters remains unknown. The existing technology for characterizing novel polyketides is based on bioactivity-guided purification, an expensive and time-consuming effort that is limited to highly abundant molecular products. 

To overcome this challenge and identify the molecular product of polyketide gene clusters in high-throughput, we present Seq2PKS, a machine learning algorithm for predicting the chemical structure of the molecular product of Type 1 polyketide gene clusters. To enhance accuracy, Seq2PKS predicts hundreds or thousands of putative structures for each gene cluster, and then the correct structure is identified among these predictions using a mass spectral database search. 

![Screenshot](image/seq2PKS_pipeline.png)

## Seq2PKS pipeline

The Seq2PKS pipeline contains five major steps:

### Detect polyketide Biosynthetic gene clusters (BGCs), Modules and Domains

This step involves using antiSMASH to identify polyketide biosynthetic gene clusters (BGCs), modules, and domains. The corresponding code is located in the **genome2genes** folder. When an NCBI ID is provided as input, Seq2PKS initially employs the esearch tool to download the relevant sequence. In cases where a sequence is directly provided, it is processed using the installed version of antiSMASH. The result from antiSMASH will be stored in the folder **antismash_result**. Once the antiSMASH analysis is complete, Seq2PKS parses the output to retain only essential information for subsequent steps. The refined output from antiSMASH is stored within the **parsed_result folder**, located in the results directory.

### Identify AT domain specificity 

In this step, a pre-trained machine learning model is employed to ascertain the AT domain specificity across all AT domains within each BGC. Relevant code for this process can be found in the **genome2genes** folder. Initially, the signature of each AT domain is extracted through alignment with a reference sequence. Subsequently, this signature is input into the model to generate the predicted results. The outcomes of this step are stored in the **specificity_result** folder.

### Predict structure order

In this step, we predict the assembly sequence of monomers recruited by each gene in the biosynthetic gene cluster (BGC). The code pertinent to this procedure is located in the **genes2domains** folder. The process begins with the extraction of the docking domain sequences from each gene. These sequences are then input into the model, which ranks all potential pathways. The ranked pathways are essential for constructing the molecular backbone. The outcomes of this step are stored in the **rank** folder.

### Construct backbone

In this step, AT domain specificity is merged with information from other domains to identify the mature monomers corresponding to each module, using a rule-based model. These monomers are then sequentially linked to create the backbone, adhering to the assembly order predicted in the preceding step. The code relevant to this process is housed in the **domains2backbone** folder. The outcomes of this step are compiled and stored in the **backbone** folder.

### Apply post-modification and score against the input spectrum file. 

In this step, post-modification genes responsible for various reactions are identified through HMM search. Subsequently, the corresponding modifications for each gene are applied to the molecular backbone using a graph-based model. The resulting candidate compounds are then scored against spectrums using tools like Dereplicator+ or Moldiscovery. The code relevant to this process is housed in the **backbone2pks** folder. The results for this step are being stored in the folder **compound**.

## Installing Seq2PKS

### Pre-built Docker image(Recommended)

This is the most straightforward way to install Seq2PKS. 

```
wget https://github.com/mohimanilab/Seq2PKS/main/docker/seq2pks.tar
sudo docker load -i seq2pks.tar
```

### Manually install on Unix-based system(Not recommended)

#### Installing Other Dependencies

If not present, you will also need to install the following:

- [ncbi-blast+](https://blast.ncbi.nlm.nih.gov/doc/blast-help/downloadblastdata.html#downloadblastdata)

For example, on Ubuntu,

```
sudo apt-get install NCBI-blast+
```

- [singularity](https://github.com/sylabs/singularity/releases)

Follow the [instructions](https://docs.sylabs.io/guides/4.0/user-guide/quick_start.html) to install it.

#### Local environment

The below packages are needed along with Python.

* `git`
* `pandas`
* `numpy`
* `rdkit`
* `antismash`
* `scikit-learn`
* `biopython`

For example, using `conda,`

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

Seq2PKS currently can take either NCBI ID or fasta file as input. Here are the input parameters for the software.

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

Here is an example of running with a FASTA file and a MZ file as input:

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
