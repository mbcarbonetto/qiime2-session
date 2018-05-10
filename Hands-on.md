---
title: "Hands-on 16S based microbial diversity analysis using QIIME2"
author: "Belen Carbonetto"
date: "May 2018"
output: html_document
---

## Hands on for 16S based microbial diversity analysis using QIIME2

### Learning outcomes:

**During this hands-on session you will learn how to:**

1- "Import" data into QIIME2 

2- Do quality filtering, denoising and picking of features and representative sequences using DADA2

3- Assign taxonomy to features with trained classifiers.

4- Align sequences and build a phylogenetic tree.

5- Calculate alpha and beta diversity.

6- Test for differential abundances between groups of samples using ANCOM

**Note:** QIIME2 has already been installed in your working station in order to follow this tutorial.

If you are interested in how to install QIIME2 on your own computer please follow [this](https://docs.qiime2.org/2018.2/install/) link.

### Core workflow:

We are going to follow this analysis workflow:

![workflow](https://github.com/mbcarbonetto/qiime2-session/blob/master/hands_on_qiime2.png)

### 0. Data set and input files

You can download the files we will use in this session from this [link](https://github.com/mbcarbonetto/qiime2-session/tree/master/files)

The test dataset we are going to use is originally from ![Batista et al. (2015)](https://www.nature.com/articles/ncomms9945#Fig5)
It is composed of ten **.fastq** files, one for each sample. Mice gut microbiota was sampled under 2 conditions: under Streptomycin treatment and with no antibiotics treatment. The data set is composed of 5 replicates for each condition. Each file consist of 10,000 subsampled reads from the original fastq files. Reads are amplicons of the V3–V4 region of the 16S rRNA gene. They are single forward reads, already demultiplexed (one file/sample), with no primers and no barcodes.

Besides the ![.fastq files](https://github.com/mbcarbonetto/qiime2-session/tree/master/files/fastq) you will find ![mapping_file.tsv](https://github.com/mbcarbonetto/qiime2-session/blob/master/files/mapping_file.tsv) This is a tab separated value table that includes metadata. The easiest way to make a mapping file is with a spreadsheet tool. However, Excel is not the best choice! It usually corrupts gene symbols, anything interpreted as dates,etc. Google Docs is prefered.
This is how the mapping file looks like:
![mapping_image](https://github.com/mbcarbonetto/qiime2-session/blob/master/mapping_file.jpg)

The column labels are always in the first row, **#** indicates that the line is not going to be read.
Sample IDs must be in the first columns, the rest of the columns include metadata. 
Sample IDs needs to be unique. 
-should be 36 characters long or less.
-should contain only ASCII alphanumeric characters (\[a-z]\, \[A-Z]\, or \[0-9]\), the period (.) character, or the dash (-) character.

Metadata columns can contain categorical or numerical values. Theay should also include labels.

You can follow this [link](https://docs.qiime2.org/2018.2/tutorials/metadata/) to learn more about metadata in QIIME2

Even if we are not doing it during this session, it is good practice to perform a validation of the mapping file.
To do that you can use Keemei, which is a  Google Sheets add-on for validating tabular bioinformatics file formats.
Instructions for installation and usage can be found [here](https://keemei.qiime2.org/)

### 1. Import data- Create an *artifact*

To start working in QIIME2 we need to activate the QIIME environment. To do so first open a terminal:

![terminal](https://github.com/mbcarbonetto/qiime2-session/blob/master/terminal_button.png)

Then type:

    source activate qiime2-2018.4

We are now creating a working directory where we are going to place the output files of this session:

    mkdir working_dir

You can also move the [input files](https://github.com/mbcarbonetto/qiime2-session/tree/master/files) you heve already dowloaded to this folder:

    mv ~/Downloads/files/ working_dir/

All data that is used as input to QIIME2 should be in form of QIIME2 artifacts, which contain information about the type of data and the source of the data. So, the first thing we need to do is import the sequence data files into a QIIME2 artifact.
The semantic type of our data is **SampleData[SequencesWithQuality]**, i.e *Sequences with quality scores (.fastq files), where each set of sequences is associated with a sample identifier (i.e. demultiplexed sequences, each file is a sample).*
In order to create the artifact with the correct metadata (source and type of data) we need a [*manifest*](https://github.com/mbcarbonetto/qiime2-session/blob/master/files/manifest.txt) file. This file will indicate the sample ID, the location of each file and the direction of the reads, in our case forward reads.

Please complete the file with the correct path for each fastq file in your computer (replace "completePATH" for the real path).

We are now ready to create a QIIME2 for our data:

    qiime tools import \
    --type 'SampleData[SequencesWithQuality]' \
    --input-path /home/Documents/working_dir/files/manifest.txt \
    --output-path /home/Documents/working_dir/single-end-demux.qza \
    --source-format SingleEndFastqManifestPhred33

With this command we are telling QIIME2 which **semantic type** (--type) to follow and what **source** (--source) of data we are importing. We are algo giving the path to the **input** fastq files with the manifest file (--input-path). Finally we are telling QIIME2 where to create the **output** file (--output-path) which will be called **single-end-demux.qza**

There are many other types of data that can be imported to QIIME2, each will have a different semantic type asigned and different protocols will be used to import them. Please follow [this link](https://docs.qiime2.org/2018.2/semantic-types/) for more information on semantic types in QIIME2 and [this one](https://docs.qiime2.org/2018.2/tutorials/importing/) to learn how to import different types of data.
    








### 2.Quality filtering, denoising and feature picking using DADA2


