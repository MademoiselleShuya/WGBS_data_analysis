# PART ONE: Installing Anaconda or Miniconda

## Download the proper and latest miniconda version, or download anaconda.
wget curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh   

## Verify data integrity with SHA-256 to make sure the correct and intact version has been downloaded
sha256sum filename

# PART TWO: Set up channels

source conda 
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge

# PART THREE: Install packages

## Personally, I would create virtual environments before installing packages, as it is much easier to manage.
conda install packagename 

# PART FOUR: Create a virtual environment

## Check current environments(env), normally there is only a base environment as long as Miniconda was just downloaded. 
conda env list

## To use this env. Now, we can download packages in the base env
source activate base

## To close this env
conda deactivate

## To create a new env for the project with the name wgbs_tools. -n means name.
conda create -n wgbs_tools

## Here is a data analysis project for whole genome bisulfate sequencing. To activate this environment, use
conda activate wgbs_tools

## Download tool packages for the project, here let us try trim-galore. Simply google conda trim-galore and get the downurls. -c means channel. 
conda install -c bioconda trim-galore

## List current packages in the new env wgbs_tools
conda env export -n wgbs_tool

# Backup the env wgbs_tools. Make a yaml file with a specific name 
conda env export > wgbs_environment.yaml

## Copy a env to make our analysis results repeatable
conda create -n XXXX -f environment.yaml

## To close this env
conda deactivate

# PART FIVE: Update and remove

## Updating Anaconda or Miniconda
conda update conda

## Uninstalling Anaconda or Miniconda
rm -rf ~/miniconda