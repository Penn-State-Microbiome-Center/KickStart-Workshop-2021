# MEGAHIT

MEGAHIT is a _de novo_ assembler first introduced in 2015 which has been continually updated since then. It utilizes mutliple k-mer sizes when building a de Bruijn graph 
along with a strategy to rescue low-coverage regions while attempting to account for sequencing error.

## Installation

MEGAHIT can be installed with conda in the following fashion
```
conda create -y -n megahit -c bioconda megahit
conda activate megahit
```

## Usage
There are a variety of parameters that can be specified with MEGAHIT:
```
megahit: MEGAHIT v1.2.9

contact: Dinghua Li <voutcn@gmail.com>

Usage:
  megahit [options] {-1 <pe1> -2 <pe2> | --12 <pe12> | -r <se>} [-o <out_dir>]

  Input options that can be specified for multiple times (supporting plain text and gz/bz2 extensions)
    -1                       <pe1>          comma-separated list of fasta/q paired-end #1 files, paired with files in <pe2>
    -2                       <pe2>          comma-separated list of fasta/q paired-end #2 files, paired with files in <pe1>
    --12                     <pe12>         comma-separated list of interleaved fasta/q paired-end files
    -r/--read                <se>           comma-separated list of fasta/q single-end files

Optional Arguments:
  Basic assembly options:
    --min-count              <int>          minimum multiplicity for filtering (k_min+1)-mers [2]
    --k-list                 <int,int,..>   comma-separated list of kmer size
                                            all must be odd, in the range 15-255, increment <= 28)
                                            [21,29,39,59,79,99,119,141]

  Another way to set --k-list (overrides --k-list if one of them set):
    --k-min                  <int>          minimum kmer size (<= 255), must be odd number [21]
    --k-max                  <int>          maximum kmer size (<= 255), must be odd number [141]
    --k-step                 <int>          increment of kmer size of each iteration (<= 28), must be even number [12]

  Advanced assembly options:
    --no-mercy                              do not add mercy kmers
    --bubble-level           <int>          intensity of bubble merging (0-2), 0 to disable [2]
    --merge-level            <l,s>          merge complex bubbles of length <= l*kmer_size and similarity >= s [20,0.95]
    --prune-level            <int>          strength of low depth pruning (0-3) [2]
    --prune-depth            <int>          remove unitigs with avg kmer depth less than this value [2]
    --disconnect-ratio       <float>        disconnect unitigs if its depth is less than this ratio times
                                            the total depth of itself and its siblings [0.1]
    --low-local-ratio        <float>        remove unitigs if its depth is less than this ratio times
                                            the average depth of the neighborhoods [0.2]
    --max-tip-len            <int>          remove tips less than this value [2*k]
    --cleaning-rounds        <int>          number of rounds for graph cleanning [5]
    --no-local                              disable local assembly
    --kmin-1pass                            use 1pass mode to build SdBG of k_min

  Presets parameters:
    --presets                <str>          override a group of parameters; possible values:
                                            meta-sensitive: '--min-count 1 --k-list 21,29,39,49,...,129,141'
                                            meta-large: '--k-min 27 --k-max 127 --k-step 10'
                                            (large & complex metagenomes, like soil)

  Hardware options:
    -m/--memory              <float>        max memory in byte to be used in SdBG construction
                                            (if set between 0-1, fraction of the machine's total memory) [0.9]
    --mem-flag               <int>          SdBG builder memory mode. 0: minimum; 1: moderate;
                                            others: use all memory specified by '-m/--memory' [1]
    -t/--num-cpu-threads     <int>          number of CPU threads [# of logical processors]
    --no-hw-accel                           run MEGAHIT without BMI2 and POPCNT hardware instructions

  Output options:
    -o/--out-dir             <string>       output directory [./megahit_out]
    --out-prefix             <string>       output prefix (the contig file will be OUT_DIR/OUT_PREFIX.contigs.fa)
    --min-contig-len         <int>          minimum length of contigs to output [200]
    --keep-tmp-files                        keep all temporary files
    --tmp-dir                <string>       set temp directory

Other Arguments:
    --continue                              continue a MEGAHIT run from its last available check point.
                                            please set the output directory correctly when using this option.
    --test                                  run MEGAHIT on a toy test dataset
    -h/--help                               print the usage message
    -v/--version                            print version
```
However, the main ones we will be concerned with are specifying if the input data (which must be `fasta` or `fastq`) is paired end (`-1` and `-2` flags), interleaved (`-12` flag) 
or `-r` single-end, as well as the specification of the output directory with `-o`.

### File directory setup and data acquisition 
As usual, we will make `data`, `scripts`, and `output` directories:
```
mkdir data scripts output
```
After this, we can use the same data that we used in the profiling section:
```
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list.txt
ls *.gz | xargs -P6 -I{} gunzip {}
```


