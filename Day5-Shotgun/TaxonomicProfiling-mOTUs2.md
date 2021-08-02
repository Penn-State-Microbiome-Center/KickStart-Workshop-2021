**mOTUs2 Tutorial**
===========================

[mOTUs2](https://motu-tool.org/index.html) is a tool for profiling the taxonomic composition of microbial communities from metagenomic
shotgun sequencing data.

This tool has a good balance of sensitivity and specificity. It is based on marker genes, but in contrast to MetaPhlAn, it uses single nucleotide variants in the marker genes
which leads to higher sensitivity.

# Installation
- Activate conda on Roar: `module load anaconda3`
- Create the environment: `conda create -y --name motus2 -c bioconda motus`
- Start the environment: `conda activate metaphlan`

# Obtaining test data:
- Make analysis folder: `mkdir mOTUs_Analysis`
- `cd mOTUs_Analysis`
- `mkdir data`
- `cd data`
- Download the data: `wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list.txt`
- Decompress the data in parallel: `ls *.gz | xargs -P6 -I{} gunzip {}`
- `cd ..`

# Create folder directory
- `mkdir output`
- `mkdir scripts`
- `cd scripts`
- `vim run_motus.sh`
