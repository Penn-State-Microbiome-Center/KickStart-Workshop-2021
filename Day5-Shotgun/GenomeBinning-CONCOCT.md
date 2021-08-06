cd CONCOCT_analysis
mkdir data scripts output output/on_MEGAHIT output/on_GATB
conda install -y mkl
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
