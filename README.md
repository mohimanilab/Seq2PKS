# Seq2PKS - Discovering modular type I Cis-AT polyketide natural products by integrating computational mass spectrometry and genome mining

## Introduction
Seq2PKS is a robust tool designed to identify novel polyketide products from input microbial genomes. By employing a series of meticulously crafted steps, it can detect biosynthetic gene clusters (BGCs), predict domain specificity, assemble monomers into backbones, apply post-modifications, and score compounds against input spectra.

The following graph represents the overall pipeline for Seq2PKS.

![Screenshot](image/seq2PKS_pipeline.png)

## Pre-request
Before running Seq2PKS, ensure that the following packages are installed:

* `pandas`
* `numpy`
* `rdkit`
* `antismash`
* `transeq`
* `sk-learn`
* `Bio`
* `blast`


## Seq2PKS pipeline
The Seq2PKS pipeline contains five major steps:
### Detect Polyketide Biosynthetic Gene Clusters (BGCs), Modules, and Domains:
This step uses antismash to detect polyketide Biosynthetic gene clusters (BGCs), Modules, and Domains. The parsed result from antismash are stored in the folder **parsed_result**
### Identify AT domain specificity 
In this step, a pre-trained machine learning model is used to predict the AT domain specificity for all AT domains in each BGC. The result for this step are being stored in the folder **specificity_result**
### Predict structure order
In this step, the assembled order of monomers being recruited by each gene in the BGC is predicted. The result for this step are being stored in the folder **rank**
### Construct backbone
In this step, The AT domain specificity is combined with other domain information to obtain the monomer corresponding to each module. These monomers are then connected to form the backbone following the order predicted in the previous step. The result for this step are being stored in the folder **backbone** 
### Apply post-modification and score against the input spectrum file
In this step, the genes that responsible for post-modification are being detected using HMM search. Then the corresponding modifications are applied to the backbone using core2pks. The candidate compounds are scored against spectrum using dereplicator+ or moldiscovery. The result for this step are being stored in the folder **compound** 


## Running Seq2PKS on computer
Seq2PKS currently can take either ncbi_id or fasta file as input. Here are the input parameters for the software.

* `ncbi_id` Ncbi_id for the input sequence
* `sequence_file` Path to the sequence file
* `sequence_id` Name for the sequence file
* `output_folder` Output folder to store the result
* `spectrum_path` Path to the spectrum file
* `pattern` Algorithm for spectrum matching can take "moldiscovery" or "dereplicator+" as input
* `n_jobs` Number of parallel jobs during computation
* `smile_compound` Expected SMILE string for the compound if known

Here is an example of running Seq2PKS with ncbi id as input:
```
python main.py \
--ncbi_id U24241.2 \
--pattern dereplicator_plus \
--spectrum_path sample_spectram.mzML \
--output_folder result
```

Here is an example of running Seq2PKS with a fasta file as input:
```
python main.py \
--sequence_file azalomycin_F.fasta \
--sequence_id azalomycin_F \
--pattern dereplicator_plus \
--spectrum_path sample_spectram.mzML \
--output_folder result
```
## Sample run
We have the sample run result included in the **test_result** folder for testing purposes. You should be able to generate the exact same result in the folder by following the below command:
```
python main.py \
--ncbi_id DQ149987.1 \
--pattern dereplicator_plus \
--spectrum_path sample_spectram.mzML \
--smile_compound 'CC=1CC(C)C(O)C(CC)C(O)C(C)C=C(C)C=C(OC)C(=O)OC(C(C(C(C3(O)CC(C(C(C=CC)O3)C)OC2OC(C(OC(=O)N)C(O)C2)C)C)O)C)C(C=CC=1)OC' \
--output_folder test_result
```
The identified compounds are stored in the file **test_result/DQ149987/compound/backbone2pks_result/database.csv**.  
The generated compounds are scored against the input spectrum file using dereplicator+, the obtained match are stored in **test_result/DQ149987/compound/backbone2pks_result/psms.tsv**.

## Seq2PKS web server

Coming soon: Seq2PKS will soon be supported as a web service at npanalysis.org
