cd CONCOCT_analysis
mkdir data scripts output output/on_MEGAHIT output/on_GATB
conda install -y mkl bwa
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt  #<<--TODO: change to single file if needed
ls *.gz | xargs -P6 -I{} gunzip {}
cp ../../MEGAHIT_analysis/output/default/final.contigs.fa MEGAHIT_default_contigs.fasta
cp ../../GATB_analysis/output/default.fasta GATB_default_contigs.fasta
# Artificially increase the length of at least two of the contigs
cd ..
conda install -y bwa
bwa index data/MEGAHIT_default_contigs.fasta
bwa mem -t 4 data/MEGAHIT_default_contigs.fasta data/SRS014464-Anterior_nares.fastq > output/on_MEGAHIT/SRS014464-Anterior_nares.sam
samtools view -S -b output/on_MEGAHIT/SRS014464-Anterior_nares.sam > output/on_MEGAHIT/SRS014464-Anterior_nares.bam
samtools sort output/on_MEGAHIT/SRS014464-Anterior_nares.bam -o output/on_MEGAHIT/SRS014464-Anterior_nares.sorted.bam
samtools index output/on_MEGAHIT/SRS014464-Anterior_nares.sorted.bam
cut_up_fasta.py data/MEGAHIT_default_contigs.fasta -c 1000 -o 0 --merge_last -b output/on_MEGAHIT/contigs_1000.bed > output/on_MEGAHIT/contigs_1000.fa
concoct_coverage_table.py output/on_MEGAHIT/contigs_1000.bed output/on_MEGAHIT/SRS014464-Anterior_nares.sorted.bam > output/on_MEGAHIT/coverage_table.tsv
concoct --composition_file output/on_MEGAHIT/contigs_1000.fa --coverage_file output/on_MEGAHIT/coverage_table.tsv -b output/on_MEGAHIT/ --threads 4
merge_cutup_clustering.py output/on_MEGAHIT/clustering_gt1000.csv > output/on_MEGAHIT/clustering_merged.csv
mkdir output/on_MEGAHIT/fasta_bins
extract_fasta_bins.py data/MEGAHIT_default_contigs.fasta output/on_MEGAHIT/clustering_merged.csv --output_path output/on_MEGAHIT/fasta_bins
# Then use CheckM or BUSCO for completeness and contamination

# Genome binning with CONCOCT

CONCOCT is a genome binning tool first introduced in 2014. While it is not the most recent or accurate tool, it serves as the starting point for a number of other tools (such   as [MetaBinner](https://www.biorxiv.org/content/10.1101/2021.07.25.453671v1), [MetaWrap](https://microbiomejournal.biomedcentral.com/articles/10.1186/s40168-018-0541-1), and [UltraBinner](https://github.com/Huangpq2019/UltraBinner)). CONCOCT uses a combination of alignment covererage information and k-mer frequecies and a Gaussian mixture model to partition the contigs in a space of dimension reduction (i.e. PCA). The following figure (obtained from [here](https://www.nature.com/articles/nmeth.3103)) depicts the different genome bins with different colors along with the mixture model used to partition them (the legend corresponds to individual genomes):

![CONCOCT](https://user-images.githubusercontent.com/6362936/128550107-4e9ad699-1221-40d4-ad88-63e74f356352.PNG)

Note that the following analysis is considered incomplete, as after running CONCOCT, you will want to run [CheckM](https://ecogenomics.github.io/CheckM/) and/or [BUSCO](https://busco.ezlab.org/) to check for conserved genes in each bin, look for contamination, etc. 

# Installation and setup

## File directory setup and data acquisition
We will use the usual file structure and copy over the data from both the GATB and MEGAHIT analysis:
```
cd CONCOCT_analysis
mkdir data scripts output output/on_MEGAHIT output/on_GATB
conda install -y mkl
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt  #<<--TODO: change to single file if needed
ls *.gz | xargs -P6 -I{} gunzip {}
cp ../../MEGAHIT_analysis/output/default/final.contigs.fa MEGAHIT_default_contigs.fasta
cp ../../GATB_analysis/output/default.fasta GATB_default_contigs.fasta
```

Note that since we are using a _very_ small demonstration sample, CONCOCT (and actually, each binning tool I tried) will say there are no bins at all. So we must artificially increase our contig lengths. The following takes each contig and copies it 5 times:
```
awk '!/^>/{next}{getline s} length(s) >= 1 { print $0 "\n" s s s s s}' data/MEGAHIT_default_contigs.fasta > data/MEGAHIT_default_contigs_longer.fasta
awk '!/^>/{next}{getline s} length(s) >= 1 { print $0 "\n" s s s s s}' data/GATB_default_contigs.fasta > data/GATB_default_contigs_longer.fasta
```

Note too that CONCOCT really would like multiple samples from the same environment to incrase accuracy (keeping track of mapping coverage per contig and sample), but we don't have that with our demo data.

## Installing CONCOCT
CONCOCT can currently be installed via the following:
```bash
conda deactivate
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda create -n concoct_env -y python=3 concoct mkl
```

## Running CONCOCT

Let's go step-by-step for the MEGAHIT assembly, and then put everything in a bash script for the GATB assembly.

### Preprocessing
CONCOCT requires a bit more manual work as we will need to create the coverage profile and k-mer spectrum before running CONCOCT.

1. **Align the reads to the assembly**

This will give us coverage information that CONCOCT will use. You can use any aligner you want, but we'll use BWA here. Recall that we did the assembly with the `SRS014464-Anterior_nares.sam`. We will need to index the contigs, do the alignment, then convert, sort, and index the resulting bam file:
```bash
bwa index data/MEGAHIT_default_contigs_longer.fasta
bwa mem -t 4 data/MEGAHIT_default_contigs_longer.fasta data/SRS014464-Anterior_nares.fastq > output/on_MEGAHIT/SRS014464-Anterior_nares.sam
samtools view -S -b output/on_MEGAHIT/SRS014464-Anterior_nares.sam > output/on_MEGAHIT/SRS014464-Anterior_nares.bam
samtools sort output/on_MEGAHIT/SRS014464-Anterior_nares.bam -o output/on_MEGAHIT/SRS014464-Anterior_nares.sorted.bam
samtools index output/on_MEGAHIT/SRS014464-Anterior_nares.sorted.bam
```

2. Cut up the contigs

CONCOCT suggest cutting your contigs into smaller, 10 Kbp chunks (so the PCA works better) but since we have such short contigs with our demo data, let's do 1 Kbp chunks instead:

```bash
cut_up_fasta.py data/MEGAHIT_default_contigs_longer.fasta -c 1000 -o 0 --merge_last -b output/on_MEGAHIT/contigs_1000.bed > output/on_MEGAHIT/contigs_1000.fa
```

3. Generate the coverage information

Now we need to keep track of where the alignments to the original contigs went when we cut things up into smaller chunks. The following script does that for us:
```bash
concoct_coverage_table.py output/on_MEGAHIT/contigs_1000.bed output/on_MEGAHIT/SRS014464-Anterior_nares.sorted.bam > output/on_MEGAHIT/coverage_table.tsv
```
Now we're finally ready to run the tool!

### Running the tool

CONCOCT itself has a variety of paramters for us to use:
```bash
$ concoct --help
usage: concoct [-h] [--coverage_file COVERAGE_FILE] [--composition_file COMPOSITION_FILE] [-c CLUSTERS] [-k KMER_LENGTH] [-t THREADS]
               [-l LENGTH_THRESHOLD] [-r READ_LENGTH] [--total_percentage_pca TOTAL_PERCENTAGE_PCA] [-b BASENAME] [-s SEED] [-i ITERATIONS]
               [--no_cov_normalization] [--no_total_coverage] [--no_original_data] [-o] [-d] [-v]

optional arguments:
  -h, --help            show this help message and exit
  --coverage_file COVERAGE_FILE
                        specify the coverage file, containing a table where each row correspond to a contig, and each column correspond to a
                        sample. The values are the average coverage for this contig in that sample. All values are separated with tabs.
  --composition_file COMPOSITION_FILE
                        specify the composition file, containing sequences in fasta format. It is named the composition file since it is used to
                        calculate the kmer composition (the genomic signature) of each contig.
  -c CLUSTERS, --clusters CLUSTERS
                        specify maximal number of clusters for VGMM, default 400.
  -k KMER_LENGTH, --kmer_length KMER_LENGTH
                        specify kmer length, default 4.
  -t THREADS, --threads THREADS
                        Number of threads to use
  -l LENGTH_THRESHOLD, --length_threshold LENGTH_THRESHOLD
                        specify the sequence length threshold, contigs shorter than this value will not be included. Defaults to 1000.
  -r READ_LENGTH, --read_length READ_LENGTH
                        specify read length for coverage, default 100
  --total_percentage_pca TOTAL_PERCENTAGE_PCA
                        The percentage of variance explained by the principal components for the combined data.
  -b BASENAME, --basename BASENAME
                        Specify the basename for files or directory where outputwill be placed. Path to existing directory or basenamewith a
                        trailing '/' will be interpreted as a directory.If not provided, current directory will be used.
  -s SEED, --seed SEED  Specify an integer to use as seed for clustering. 0 gives a random seed, 1 is the default seed and any other positive
                        integer can be used. Other values give ArgumentTypeError.
  -i ITERATIONS, --iterations ITERATIONS
                        Specify maximum number of iterations for the VBGMM. Default value is 500
  --no_cov_normalization
                        By default the coverage is normalized with regards to samples, then normalized with regards of contigs and finally log
                        transformed. By setting this flag you skip the normalization and only do log transorm of the coverage.
  --no_total_coverage   By default, the total coverage is added as a new column in the coverage data matrix, independently of coverage
                        normalization but previous to log transformation. Use this tag to escape this behaviour.
  --no_original_data    By default the original data is saved to disk. For big datasets, especially when a large k is used for compositional
                        data, this file can become very large. Use this tag if you don't want to save the original data.
  -o, --converge_out    Write convergence info to files.
  -d, --debug           Debug parameters.
  -v, --version         show program's version number and exit
```

Of note are the following parameters: `--clusters <num_clusters>` this specifies the maximum number of clusters CONCOCT will find. The default is 400, so be aware if you have many deep, highly diverse samples. `--length_threshold <len>` specifies which contigs are to be ignored since they are "too short" and is by default set to 1Kbp.  The `--kmer_ length <size>` and `--total_percentage_pca` will both affect how many different bins you end up with in the end (eg. with certain settings here, you can merge everything into one bin, or have each contig belong to its own bin).

Let's go ahead an just use the default settings (always a good starting point):
```
concoct --composition_file output/on_MEGAHIT/contigs_1000.fa --coverage_file output/on_MEGAHIT/coverage_table.tsv -b output/on_MEGAHIT/ --threads 4
```