# Seq2PKS

## Introduction
Seq2PKS is aimed to identify novel polyketide products based on input microbial genome.

The below graph is the overall pipeline for Seq2PKS.

![Screenshot](image/seq2pks_overview.png)

## Pre-requist
In order to run Seq2PKS, below packages is needed.
* `pandas`
* `numpy`
* `rdkit`
* `antismash`
* `transeq`
* `sk-learn`

## Seq2PKS pipeline
The Seq2PKS pipeline contains five major steps:
### Detect polyketide Biosynthetic gene clusters (BGCs), Modules and Domains
This step use antismash to detect polyketide Biosynthetic gene clusters (BGCs), Modules and Domains. The parsed result from antismash are stored in the folder **parsed_result**
### Identify AT domain specificity 
In this step, a pre-trained machine learning model is used to predict the AT domain specificity for all AT domain in each BGC. The result for this step are being stored in the folder **specificity_result**
### Predict structure order
In this step, the assemble order for monomers being recruited by each gene in the BGC is being predicted. The result for this step are being sotred in the folder **rank**
### Construct backbone
In this step, The AT domain specificity are combined with other domain information to obtain the monomer correspond to each module. These monomers are then being connected to form the backbone following the order predicted in the previous step. The result for this step are being sotred in the folder **backbone** 
### apply post-modification and score against input spectrum file. 
In this step, the genes that responsible for post-modification are being detected using HMM search. Then the corresponding modifications are being applied to the backbone using core2pks. The candidate compounds are scored against spectrum using dereplicator+ or moldiscovery. The result for this step are being sotred in the folder **compound** 


## Running Seq2PKS on computer
Seq2PKS currently can take either Ncbi_id or fasta file as input. Here are the input parameters for the software.

* `ncbi_id` ncbi_id for the input sequence
* `sequence_file` path to the sequence file
* `sequence_id` name for the sequence file
* `output_folder` output folder to store the result
* `spectrum_path` path to the spectrum file
* `pattern` algorithm for spectrum matching, can take "moldiscovery" or "dereplicator+" as input
* `n_jobs` Number of parallel jobs during computation

Here is the sample code for running Seq2PKS with ncbi id as input:
```
python main.py \
--ncbi_id U24241.2 \
--pattern dereplicator+ \
--spectrum_path PDorr_0045_14Sep12_Lynx_12-06-05.mzXML \
--output_folder result
```

Here is the sample code for running Seq2PKS with fasta file as input:
```
python main.py \
--sequence_file azalomycin_F.fasta \
--sequence_id azalomycin_F \
--pattern dereplicator+ \
--spectrum_path PDorr_0045_14Sep12_Lynx_12-06-05.mzXML\
--output_folder result
```


And some optional parameters inside the function

* `maxdepth`  The maximum depth of post-modifications applied on backbones
* `maxspon` The maximum number of primary reactions to be considered
* `maxringsadd` The maximum number of 6-member or 5-member rings applied on backbones
* `mode` 3 different mode to apply post-modifications, mode 0 input all possible modifications to the dereplicator software, mode 2 add some restriction on the basis of mode 0

Here is the default number of these optional parameters

```
maxdepth 5
maxspon 2
maxringsadd 2
mode 0
```

## Seq2PKS web server

Coming soon: Seq2PKS will soon be supported as a web service at npanalysis.org
