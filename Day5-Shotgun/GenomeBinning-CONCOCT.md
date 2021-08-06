cd CONCOCT_analysis
mkdir data scripts output output/on_MEGAHIT output/on_GATB
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list_fastq.txt  #<<--TODO: change to single file if needed
ls *.gz | xargs -P6 -I{} gunzip {}
cp ../../MEGAHIT_analysis/output/default/final.contigs.fa MEGAHIT_default_contigs.fasta
cp ../../GATB_analysis/output/default.fasta GATB_default_contigs.fasta
cd ..
conda install -y bwa
bwa index data/MEGAHIT_default_contigs.fasta
bwa mem -t 4 data/MEGAHIT_default_contigs.fasta data/SRS014464-Anterior_nares.fastq > data/SRS014464-Anterior_nares.sam
samtools view -S -b data/SRS014464-Anterior_nares.sam > data/SRS014464-Anterior_nares.bam
samtools sort data/SRS014464-Anterior_nares.bam -o data/SRS014464-Anterior_nares.sorted.bam
samtools index data/SRS014464-Anterior_nares.sorted.bam
cut_up_fasta.py data/MEGAHIT_default_contigs.fasta -c 100 -o 0 --merge_last -b output/on_MEGAHIT/contigs_100.bed > output/on_MEGAHIT/contigs_100.fa
concoct_coverage_table.py output/on_MEGAHIT/contigs_100.bed data/SRS014464-Anterior_nares.sorted.bam > output/on_MEGAHIT/coverage_table.tsv
concoct --composition_file output/on_MEGAHIT/contigs_100.fa --coverage_file output/on_MEGAHIT/coverage_table.tsv -b output/on_MEGAHIT/
