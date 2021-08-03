**mOTUs2 Tutorial**
===========================

[mOTUs2](https://motu-tool.org/index.html) is a tool for profiling the taxonomic composition of microbial communities from metagenomic
shotgun sequencing data.

This tool has a good balance of sensitivity and specificity. It is based on marker genes, but in contrast to MetaPhlAn, it uses single nucleotide variants in the marker genes
which leads to higher sensitivity.

# Installation
- Activate conda on Roar: `module load anaconda3`
- Create the environment: `conda create -y --name motus2 -c bioconda motus`
- Start the environment: `conda activate motus2`

# Obtaining test data:
- Make analysis folder: `mkdir mOTUs_Analysis`
- `cd mOTUs_Analysis`
- `mkdir data`
- `cd data`
- Download the data: `wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt`
- Decompress the data in parallel: `ls *.gz | xargs -P6 -I{} gunzip {}`
- `cd ..`

# Create folder directory
- `mkdir output`
- `mkdir scripts`
- `cd scripts`
- `vim run_motus.sh`

------------------------------------------------------------------------
**Create taxonomic profiles**
-----------------------------
mOTUs2 accepts as input short reads from a single shotgun
metagenomic sequencing experiment and outputs the list of detected
microbes and their relative abundances. 

### **Input files**

MetaPhlAn accepts metagenomic sequences in `.fastq`, `.sam`, or `.bam` formats.

mOTUs2 has a few different modes of operation, see these modes with `motus -h`:
```
Usage: motus <command> [options]

Command:
 -- Taxonomic profiling
      profile     Perform a taxonomic profiling (map_tax + calc_mgc + calc_motu)
      merge       Merge different taxonomic profiles to create a table

      map_tax     Map reads to the marker gene database
      calc_mgc    Aggregate reads from the same marker gene cluster (mgc)
      calc_motu   From a mgc abundance table, produce the mOTU abundance table

 -- SNV calling
      map_snv     Map reads to create bam/sam file for snv calling
      snv_call    SNV calling using metaSNV
```
We will first focus on the profiling ability of mOTUs: `motus profile -h`:
```
Usage: motus profile [options]

Input options:
   -f FILE[,FILE]  input file(s) for reads in forward orientation, fastq formatted
   -r FILE[,FILE]  input file(s) for reads in reverse orientation, fastq formatted
   -s FILE[,FILE]  input file(s) for reads without mate, fastq formatted
   -n STR          sample name
   -i FILE[,FILE]  provide sam or bam input file(s)
   -m FILE         provide a mgc reads count file

Output options:
   -o FILE         output file name [stdout]
   -I FILE         save the result of bwa in bam format (intermediate step) [None]
   -M FILE         save the mgc reads count (intermediate step) [None]
   -e              profile only reference species (ref_mOTUs)
   -c              print result as counts instead of relative abundances
   -p              print NCBI id
   -u              print the full name of the species
   -q              print the full rank taxonomy
   -B              print result in BIOM format
   -C STR          print result in CAMI format (BioBoxes format 0.9.1)
                   Values: [precision, recall, parenthesis]
   -k STR          taxonomic level [mOTU]
                   Values: [kingdom, phylum, class, order, family, genus, mOTU]

Algorithm options:
   -g INT          number of marker genes cutoff: 1=higher recall, 10=higher precision [3]
   -l INT          min. length of alignment for the reads (number of nucleotides) [75]
   -t INT          number of threads [1]
   -v INT          verbose level: 1=error, 2=warning, 3=message, 4+=debugging [3]
   -y STR          type of read counts [insert.scaled_counts]
                   Values: [base.coverage, insert.raw_counts, insert.scaled_counts]
```

For our purposes, the most important parameter is `-C` which specifies the CAMI format. There are three different choices:
```
-C precision    # higher precision
-C recall       # higher recall
-C parenthesis  # a balance between precision and recall
```.

Since we now have practice running multiple samples, let's jump straight to:

### **Run multiple samples**

Let's run, in parallel, all the samples through mOTUs2:
```
#!/bin/bash
set -e  # exit if there is an error
set -u  # exit if a variable is undefined

baseFolder="/home/dmk333/KickStartWorkshop2021/mOTUs_Analysis"  #<<---- replace with your base directory
outputFolder="${baseFolder}/output"

# Now analyze everything in one go
inputFolder="${baseFolder}/data/"
for file in `ls ${inputFolder}/*.fastq`;
do
	baseName=$(basename $file)
	motus profile -s ${file} -o ${outputFolder}/${baseName%.fastq}.profile -t 5 &
done
```
