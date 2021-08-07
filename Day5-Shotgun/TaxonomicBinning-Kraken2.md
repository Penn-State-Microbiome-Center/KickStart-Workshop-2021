# Taxonomic binning with Kraken2

Kraken is a very widely utilized metagenomics tool (dating back to 2014) and Kraken2 is its most recent iteration that boasts higher accuracy and decreased system resource utilization over Kraken1. 

In contrast to mOTUs2 and MetaPhlAn3 which use clade specific marker _genes_, Kraken2 utilizes clade specific marker _kmers_. This is summarized in the first figure of the [Kraken1 manuscript](https://genomebiology.biomedcentral.com/articles/10.1186/gb-2014-15-3-r46):

![Kraken](https://user-images.githubusercontent.com/6362936/128582419-7da7594e-9483-4138-859e-d0f0e332bad3.PNG)

In using k-mers, Kraken2 has the ability to be blazingly fast, but longer query sequences are required to account for the inherent noise when utilizing k-mers in the size ranges that Kraken utilizes. Probably due to it's relatively early implementation and how fast the tool is, Kraken has been used in quite a few situation it should not have. For example, remember when we covered [taxonomic profiling](TaxonomicProfiling.md)? Well, taxonomic profiling doesn't seem _too_ different from taxonomic binning, right? A sequence is a sequence, right? Unfortunately, Kraken is quite sensitive to sequence length (as well as sequencing error). So many people were (incorrectly) using Kraken for taxonomic profiling that the authors of Kraken were motivated to publish [Bracken](https://peerj.com/articles/cs-104/) which is an extension of Kraken designed for taxonomic profiling

```
conda deactivate
conda create -y -n kraken2 kraken2
conda activate kraken2
mkdir Kraken2_analysis
cd Kraken2_analysis
mkdir data data/k2train8gb scripts output
cd data/k2train8gb
wget https://genome-idx.s3.amazonaws.com/kraken/k2_standard_8gb_20210517.tar.gz # Via https://github.com/BenLangmead/aws-indexes/blob/master/docs/k2.md
tar -xzvf k2_standard_8gb_20210517.tar.gz
cd ../..
kraken2 --db . --threads 10 --output kraken_out.txt --classified-out kraken_cl_out.txt --use-names anonymous_reads.fq
```
