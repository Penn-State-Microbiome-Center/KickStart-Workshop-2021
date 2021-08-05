# GATB
GATB, aka The Genome Analysis Toolbox with de-Bruijn Graph is a quite flexible toolkit of various genomic analysis tools. check out their [software](https://gatb.inria.fr/software/) 
page if you would like to count k-mers in, error correct, assemble, compare, etc. genomic and metagenomic samples. One of the main authors of this toolkit is Rayan Chikhi, a 
former postdoc of Paul Medvedev's (CSE & BMB)!

While there are a lot of different goodies we could look at, we will focus just on their pipeline for _de novo_ metagenome assembly which combines the tools of 
[Bloocoo](https://gatb.inria.fr/software/bloocoo/) error correction, [Minia](https://github.com/ksahlin/BESST) assembly, and [BESST](https://gatb.inria.fr/software/minia/) scaffolding.

## Installation and setup
Let's use the usual folder structure
```
mkdir GATB_analysis
cd GATB_analysis
mkdir scripts output data
```

This pipeline is provided as a stand-along binary that can be installed via the following
```
cd scripts
wget http://gatb-pipeline.gforge.inria.fr/versions/bin/gatb-pipeline-1.171.tar.gz
tar -xzvf gatb-pipeline-1.171.tar.gz
```
As oposed to the previous tools, this one is not installed with conda, so we will need to specify exactly where the program is whenever we run it.
The main way we invoke this pipeline is the executable `gatb` which will be invoked with Python2.

Lastly, we will again obtain the (same) data:
```
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list.txt
ls *.gz | xargs -P6 -I{} gunzip {}
cd ..
```

## Running GATB

To run the GATB pipeline with default settings, we can do the following
