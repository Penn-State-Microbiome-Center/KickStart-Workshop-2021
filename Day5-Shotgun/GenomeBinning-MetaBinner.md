# Genome binning with MetaBinner

MetaBinner is an ensemble-based approach to genome binning, originating in 2019. The basic workflow can be found in the [associated preprint](https://www.biorxiv.org/content/10.1101/2021.07.25.453671v1):
![MetaBinner](https://user-images.githubusercontent.com/6362936/128402144-07f5135e-d36f-4cc7-b2eb-a6e6f1019919.PNG)

# Installation and setup

## File directory setup and data acquisition
We will use the usual file structure and data:
```
mkdir MetaBinner_analysis
cd MetaBinner_analysis
mkdir data scripts output
cd data
wget -i https://raw.githubusercontent.com/Penn-State-Microbiome-Center/KickStart-Workshop-2021/main/Day5-Shotgun/Data/file_list.txt
ls *.gz | xargs -P6 -I{} gunzip {}
cd ..
```

## Installing MetaBinner
MetaBinner can currently be installed via the following:
```bash
conda deactivate
cd scripts
git clone https://github.com/ziyewang/MetaBinner.git
cd MetaBinner
conda env create -f metabinner_env.yaml
conda activate metabinner_env  #<<-- note that we did not give this name to the environment; the name is contained in the yaml file itself.
conda install -y -c bioconda prodigal hmmer pplacer
cd CheckM-1.0.18
python setup.py install
mkdir checkm_data
cd checkm_data
wget https://data.ace.uq.edu.au/public/CheckM_databases/checkm_data_2015_01_16.tar.gz
tar -xzf checkm_data_2015_01_16.tar.gz 
checkm data setRoot .
cd ../..
```
Please note that this is a Linux-specific installation, but the developers are [working](https://github.com/ziyewang/MetaBinner/issues/4) on a cross-platform installation method (i.e. conda).
