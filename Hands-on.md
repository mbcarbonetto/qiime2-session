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

3- Align sequences and build a phylogenetic tree.

4- Calculate alpha and beta diversity.

5- Assign taxonomy to features with trained classifiers.

6- Test for differential abundances between groups of samples using ANCOM

**Note:** QIIME2 has already been installed in your working station in order to follow this tutorial.

If you are interested in how to install QIIME2 on your own computer please follow [this](https://docs.qiime2.org/2018.2/install/) link.

### Core workflow:

We are going to follow this analysis workflow:

![workflow](https://github.com/mbcarbonetto/qiime2-session/blob/master/hands_on_qiime2.png)

**Note:** every task that you will perfom is going to be marked with this symbol &#x1F536;

   Questions to further analyse results will be marked with this symbol :question:

### 0. Data set and input files

&#x1F536; You will need to download the files we will use in this session from this [link](https://github.com/mbcarbonetto/qiime2-session/tree/master/files)

The test dataset we are going to use is originally from ![Batista et al. (2015)](https://www.nature.com/articles/ncomms9945#Fig5)
It is composed of ten **.fastq** files, one for each sample. Mice gut microbiota was sampled under 2 conditions: under Streptomycin treatment and with no antibiotics treatment. The data set is composed of 5 replicates for each condition. Each file consist of 10,000 subsampled reads from the original fastq files. Reads are amplicons of the V3–V4 region of the 16S rRNA gene. They are single forward reads, already demultiplexed (one file/sample), with no primers and no barcodes.

Besides the ![.fastq files](https://github.com/mbcarbonetto/qiime2-session/tree/master/files/fastq) you will find ![mapping_file.tsv](https://github.com/mbcarbonetto/qiime2-session/blob/master/files/mapping_file.tsv) This is a tab separated value table that includes metadata. The easiest way to make a mapping file is with a spreadsheet tool. However, Excel is not the best choice! It usually corrupts gene symbols, anything interpreted as dates,etc. Google Docs is prefered.
This is how the mapping file looks like:
![mapping_image](https://github.com/mbcarbonetto/qiime2-session/blob/master/mapping_file.jpg)

The column labels are always in the first row, **#** indicates that the line is not going to be read as a sample.
Sample IDs must be in the first column, the rest of the columns include metadata. 
Sample IDs needs to be unique. 
-should be 36 characters long or less.
-should contain only ASCII alphanumeric characters (\[a-z]\, \[A-Z]\, or \[0-9]\), the period (.) character, or the dash (-) character.

Metadata columns can contain categorical or numerical values. They should also include labels.

You can follow this [link](https://docs.qiime2.org/2018.2/tutorials/metadata/) to learn more about metadata in QIIME2

Even if we are not doing it during this session, it is good practice to perform a validation of the mapping file.
To do that you can use Keemei, which is a  Google Sheets add-on for validating tabular bioinformatics file formats.
Instructions for installation and usage can be found [here](https://keemei.qiime2.org/)

### 1. Import data- Create an *artifact*

&#x1F536; To start working in QIIME2 we need to activate the QIIME environment. To do so first open a terminal:

![terminal](https://github.com/mbcarbonetto/qiime2-session/blob/master/terminal_button.png)

&#x1F536; Then type:

    source activate qiime2-2018.4

&#x1F536; We are now creating a working directory where we are going to place the output files of this session:

    mkdir working_dir

&#x1F536; You can also move the [input files](https://github.com/mbcarbonetto/qiime2-session/tree/master/files) you heve already dowloaded to this folder:

    mv ~/Downloads/files/ working_dir/

All data that is used as input to QIIME2 should be in form of QIIME2 artifacts, which contain information about the type of data and the source of the data. So, the first thing we need to do is import the sequence data files into a QIIME2 artifact.
The semantic type of our data is **SampleData[SequencesWithQuality]**, i.e *Sequences with quality scores (.fastq files), where each set of sequences is associated with a sample identifier (i.e. demultiplexed sequences, each file is a sample).*
In order to create the artifact with the cointerrrect metadata (source and type of data) we need a [*manifest*](https://github.com/mbcarbonetto/qiime2-session/blob/master/files/manifest.txt) file. This file will indicate the sample ID, the location of each file and the direction of the reads, in our case forward reads.

&#x1F536; Please complete the file with the correct path for each fastq file in your computer (replace "completePATH" for the real path).

&#x1F536; We are now ready to create a QIIME2 artifact with our data:

    qiime tools import \
    --type 'SampleData[SequencesWithQuality]' \
    --input-path /home/Documents/working_dir/files/manifest.txt \
    --output-path /home/Documents/working_dir/single-end-demux.qza \
    --source-format SingleEndFastqManifestPhred33

With this command we are telling QIIME2 which **semantic type** (--type) to follow and what **source** (--source) of data we are importing. We are algo giving the path to the **input** fastq files with the manifest file (--input-path). Finally we are telling QIIME2 where to create the **output** file (--output-path) which will be called **single-end-demux.qza**

There are many other types of data that can be imported to QIIME2, each will have a different semantic type asigned and different protocols will be used to import them. Please follow [this link](https://docs.qiime2.org/2018.2/semantic-types/) for more information on semantic types in QIIME2 and [this one](https://docs.qiime2.org/2018.2/tutorials/importing/) to learn how to import different types of data.

Note: QIIME2 has a different argument structure than the previous QIIME version. We need to write first the command **qiime**, then the **plugin** name, then the **method** name and then the **arguments** (input, output and parameters). So, we need to follow this structure (the order is not important):

    qiime plugin method \
    --i-<input>
    --m-<metadata>
    --p-<parameter1>
    --p-<parameter2>
    ...
    --o-<output>

&#x1F536; After creating the artifact **single-end-demux.qza** we can create a viszaualition file to explore our data:

    qiime demux summarize \
    --i-data single-end-demux.qza \
    --o-visualization single-end-demux.qzv

All QIIME2 visualizers (i.e., commands that take a --o-visualization parameter) will generate a **.qzv** file. You can view these files with **qiime tools view**. 

    qiime tools view single-end-demux.qzv

This visualization allows to explore descriptive statistics of the sample sizes (i.e. min, max, median, mean, histogram) and samples quality based on Quality Score per base.

For our data set, the **Overvirew** information is not very informative since we know beforehand that all samples have the same size (10,000 reads). However, take a look at que **Interactive Quality Plot**:

:question: what is the read size when quality falls below Q20?

### 2.Quality filtering, denoising and feature picking using DADA2

We are now ready to perform quality control and feature picking. We are going to use DADA2. 
This method is based on correcting (where possible) Illumina sequencing mistakes. Features (also called amplicon sequence variants or ASV)  are inferred by a de novo process in which biological (true) sequences are discriminated from errors on the basis of the expectation that biological sequences are more likely to be repeatedly observed than are error-containing sequences.
You can get further details on the method [here](https://www.nature.com/articles/nmeth.3869) and [here](https://www.biorxiv.org/content/early/2015/08/06/024034).

&#x1F536; Run the following command in order to apply DADA2 on **single-end-demux.qza**:

    qiime dada2 denoise-single \
    --i-demultiplexed-seqs single-end-demux.qza \
    --p-trunc-len 0 \
    --p-trunc-q 19 \
    --output-dir DADA2

This command may take up to 30 minutes to run.

&#x1F536; While you wait open a new terminal, open the QIIME2 environment and read the specifiactions of the parameters we have just used:

    source activate qiime2-2018.4
    qiime dada2 denoise-single -- help
    
:question:

- Which criteria are we using to filter data by quality?

- Could have we used another filtering/trimming method? (hint: take a look at the **Interactive Quality Plot**)

- Are we remmoving chimeras? If so, which method are we using?

- What are you expecting as output files?

&#x1F536; You can now crate a summary of the results and visualize it using *qiime tools view**

    qiime feature-table summarize \
    --i-table DADA2/table.qza \
    --o-visualization DADA2/table-dada2.qzv \
    --m-sample-metadata-file /home/Documents/working_dir/files/mapping_file.tsv
    
    qiime tools view DADA2/table-dada2.qzv

:question: Explore results:

- How many features has DADA2 resolved?
- How many reads remain after quality filtering?
- Which sample has the minimum frequency of reads? and the maximum?

We have now the feature table: **table.qza**, and we have assigned a representative sequence for each feature: **representative_sequences.qza**.
Before we continue with the analysis we can export the feature table son we can explore data with another software if needed.

&#x1F536; In order to export **table.qza** run:

    qiime tools export DADA2/table.qza --output-dir exported_table
    cd exported_table
    biom convert -i feature-table.biom -o feature-table.txt --to-tsv
    cd ..
    
We have just converted the feature table **table.qza** into **feature-table.biom**, which is useful for analysis with [Picrust](http://picrust.github.io/picrust/) or [Phyloseq](https:/qiime diversity core-metrics-phylogeneticjoey711.github.io/phyloseq/) for example, and **feature-table.txt** which can be visualized in any spreadsheet app.

### 3- Align sequences and build a phylogenetic tree.

Later we are going to calculate several diversity metrics and distances that will include phylogenetic information of the features (i.e. Unifrac and Faith’s Phylogenetic Diversity). In this step we are going to align the representative sequences of each feature and then buid a phylogenetic tree. This requires four steps: the alignment itself, denoising of highly variable positions and builidng and rooting the pylogenetic tree.

&#x1F536; Step 1: Alignment

    qiime alignment mafft \
    --i-sequences DADA2/representative_sequences.qza \
    --o-alignment aligned-rep-seqs-dada2.qza

&#x1F536; Step 2: Filter alignment

    qiime alignment mask \
    --i-alignment aligned-rep-seqs-dada2.qza \
    --o-masked-alignment masked-aligned-rep-seqs-dada2.qza
    
&#x1F536; Step 3: Buid the tree

    qiime phylogeny fasttree \
    --i-alignment masked-aligned-rep-seqs-dada2.qza \
    --o-tree unrooted-tree.qza
  
&#x1F536; Step 4: Root the tree

    qiime phylogeny midpoint-root \
    --i-tree unrooted-tree.qza \
    --o-rooted-tree rooted-tree.qza
    
These commands will not generate any visulization otuput.

Now that we have the phylogenetic tree we are ready to calculate diversity.

### 4- Calculate alpha and beta diversity

Using a single command we are going to calculate the following metrics:

**Alpha	diversity**
- *Shannon’s	diversity	index*	(a quantitative	measure	of community richness)
- *Observed	OTUs*	(just community	richness)
- *Faith’s Phylogenetic Diversity* (a qualitiative measure of community	richness that incorporates phylogenetic	relationships between the features)
- *Evenness*	(or	Pielou’s Evenness;	a measure of community evenness)

**Beta	diversity**
- *Jaccard distance*	(a	qualitative	measure	of community dissimilarity)
- *Bray-Curtis	distance*	(a	quantitative measure of	community dissimilarity)
- *Unweighted UniFrac distance*	(a qualitative measure of community	dissimilarity that incorporates	phylogenetic relationships between	the	features)
- *Weighted	UniFrac distance* (a quantitative measure of community dissimilarity that incorporates	phylogenetic relationships	between	the	features.

&#x1F536; Run the following command to calculate diveristy:

      qiime diversity core-metrics-phylogenetic \ 
      --i-phylogeny rooted-tree.qza \ 
      --i-table DADA2/table.qza \ 
      --p-sampling-depth 5474 \
      --m-metadata-file /home/Documents/working_dir/files/mapping_file.tsv \
      --output-dir core-metrics-results

:question: Can you tell why **--p-sampling-depth** was set to **5474**?

This value was chosen based on the number of sequences in the **WT.day3.15** which is the one sample that has *fewer sequences*. Because most diversity metrics are sensitive to different sampling depths across different samples,we need to normalize data based on some criterium. Sub-sampling without replacement up to minimum sample size is one way to deal with this. This is probably not the best solution, but the most frequently used and accepted so far.

This script will randomly subsample the counts from each sample to the value provided for this parameter. For example, if you provide --p-sampling-depth 500, this step will subsample the counts in each sample without replacement so that each sample in the resulting table has a total count of 500. If the total count for any sample(s) are smaller than this value, those samples will be dropped from the diversity analysis. Choosing this value is tricky. You can make this choice by reviewing the information presented in the **table.qzv** file that was created above and choosing a value that is as high as possible (so you retain more sequences per sample) while excluding as few samples as possible.

Several visualizarion files were created, you will have one **.qzv** file for each beta diveristy metric that was calculated. 

&#x1F536; You can explore results using **qiime tools view**

    qiime tools view core-metrics-results/unweighted_unifrac_emperor.qzv

&#x1F536; Take a look at the results for all beta diveristy distances. (Take advantage of the interactive functionalities)

:question: Can you observe any pattern or grouping of samples?

We can now test for the significance of these grouping by using the following command:

    qiime diversity beta-group-significance \ 
    --i-distance-matrix unweighted_unifrac_distance_matrix.qza \ 
    --m-metadata-file /home/Documents/working_dir/files/mapping_file.tsv \ 
    --m-metadata-column AntibioticUsage \ 
    --o-visualization unweighted_unifrac_AntibioticUsage-significance.qzv









