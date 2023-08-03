# Seq2PKS - Identifying Novel Polyketide Products from Microbial Genomes

# Introduction
# Seq2PKS is a tool designed to identify novel polyketide products based on input microbial genomes. The pipeline employs various steps to analyze the genomic data and predict polyketide biosynthetic pathways.

# The graph below illustrates the overall Seq2PKS pipeline:

![Screenshot](image/seq2pks_overview.png)

# Prerequisites
# Before running Seq2PKS, the following packages are required:
# * `pandas`
# * `numpy`
# * `rdkit`
# * `antismash`
# * `transeq`
# * `sk-learn`

# Seq2PKS Pipeline
# The Seq2PKS pipeline consists of five major steps:

# Step 1: Detect Polyketide Biosynthetic Gene Clusters (BGCs), Modules, and Domains
# This step utilizes antismash to detect polyketide biosynthetic gene clusters (BGCs), modules, and domains. The parsed results from antismash are stored in the **parsed_result** folder.

# Step 2: Identify AT Domain Specificity
# Here, a pre-trained machine learning model predicts the specificity of the acyltransferase (AT) domain for each BGC. The results for this step are stored in the **specificity_result** folder.

# Step 3: Predict Structure Order
# The pipeline predicts the assembly order of monomers recruited by each gene in the BGC. The results are stored in the **rank** folder.

# Step 4: Construct Backbone
# The AT domain specificities are combined with other domain information to determine the monomers corresponding to each module. These monomers are then connected to form the backbone following the order predicted in the previous step. The results are stored in the **backbone** folder.

# Step 5: Apply Post-modification and Score Against Input Spectrum File
# In this step, genes responsible for post-modifications are detected using HMM search. The corresponding modifications are then applied to the backbone using core2pks. The candidate compounds are scored against the input spectrum using either dereplicator+ or moldiscovery. The results are stored in the **compound** folder.

# Running Seq2PKS on Your Computer
# Seq2PKS can take either an NCBI ID or a fasta file as input. Here are the input parameters for the software:

# * `ncbi_id`: NCBI ID for the input sequence
# * `sequence_file`: Path to the sequence file
# * `sequence_id`: Name for the sequence file
# * `output_folder`: Output folder to store the result
# * `spectrum_path`: Path to the spectrum file
# * `pattern`: Algorithm for spectrum matching, can take "moldiscovery" or "dereplicator+" as input
# * `n_jobs`: Number of parallel jobs during computation

# Here is an example code for running Seq2PKS with an NCBI ID as input:
# ```bash
# python main.py \
# --ncbi_id U24241.2 \
# --pattern dereplicator+ \
# --spectrum_path PDorr_0045_14Sep12_Lynx_12-06-05.mzXML \
# --output_folder result
# ```

# And here is an example code for running Seq2PKS with a fasta file as input:
# ```bash
# python main.py \
# --sequence_file azalomycin_F.fasta \
# --sequence_id azalomycin_F \
# --pattern dereplicator+ \
# --spectrum_path PDorr_0045_14Sep12_Lynx_12-06-05.mzXML \
# --output_folder result
# ```

# Additionally, there are some optional parameters that can be used inside the function:

# * `maxdepth`: The maximum depth of post-modifications applied to backbones
# * `maxspon`: The maximum number of primary reactions to be considered
# * `maxringsadd`: The maximum number of 6-member or 5-member rings applied to backbones
# * `mode`: Three different modes to apply post-modifications, where mode 0 inputs all possible modifications to the dereplicator software, and mode 2 adds some restrictions based on mode 0.

# The default values for these optional parameters are as follows:
# ```bash
# maxdepth 5
# maxspon 2
# maxringsadd 2
# mode 0
# ```

# Seq2PKS Web Server
# Coming soon: Seq2PKS will soon be supported as a web service at npanalysis.org





