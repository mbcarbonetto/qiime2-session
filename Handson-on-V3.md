---
title: "Hands-on 16S based microbial diversity analysis using QIIME2"
author: "Belen Carbonetto"
date: "October 2018"
Version 3
output: html_document
---

## Hands on for 16S based microbial diversity analysis using QIIME2

### Learning outcomes:

**During this hands-on session you will learn how to:**

1- "Import" data into QIIME2 

2- Do quality filtering, denoising and picking of features and representative sequences (ASV) using DADA2

3- Align sequences and build a phylogenetic tree.

4- Calculate alpha and beta diversity.

5- Assign taxonomy to features with trained classifiers.

6- Test for differential abundances between groups of samples using ANCOM

**Note:** QIIME2 has already been installed in your working station in order to follow this tutorial.

If you are interested in how to install QIIME2 on your own computer please follow [this](https://docs.qiime2.org/2018.8/install/) link.

### Core workflow:

We are going to follow this analysis workflow:

![workflow](https://github.com/mbcarbonetto/qiime2-session/blob/master/annexed_files/hands_on_qiime2.png)

**Note:** 
- Every task that you will perfom is marked with &#x1F536;

- Questions to further analyse results will be marked with :question:

### 0. Data set and input files

&#x1F536; You will need to download the files we will use in this session from this [link](https://github.com/mbcarbonetto/qiime2-session/blob/master/hands_on_files.zip). 

&#x1F536; You will need to unzip the file:

    unzip ~/Downloads/hands_on_files.zip -d hands_on_files

The test dataset we are going to use is originally from [Batista et al. (2015)](https://www.nature.com/articles/ncomms9945).
It is composed of ten **.fastq** files, one for each sample. Mice gut microbiota was sampled under 2 conditions: under Streptomycin treatment and with no antibiotics treatment. The data set is composed of 5 replicates for each condition. Each file consist of 10,000 subsampled reads from the original fastq files. Reads are amplicons of the V3–V4 region of the 16S rRNA gene. They are single forward reads, already demultiplexed (one file/sample), with no primers and no barcodes.

Besides the *.fastq* files you will find *mapping_file.tsv*. This is a tab separated value table that includes metadata. The easiest way to make a mapping file is with a spreadsheet tool. However, Excel is not the best choice! It usually corrupts gene symbols, anything interpreted as dates,etc. Google Docs is prefered.

This is how the mapping file looks like:

![mapping_image](https://github.com/mbcarbonetto/qiime2-session/blob/master/annexed_files/mapping_view.jpg)

The column labels are always in the first row, **#** indicates that the line is not going to be read as a sample.
Sample IDs must be in the first column, the rest of the columns include metadata. 
Sample IDs needs to be unique and: 
- should be 36 characters long or less.
- should contain only ASCII alphanumeric characters (\[a-z]\, \[A-Z]\, or \[0-9]\), the period (.) character, or the dash (-) character.

Metadata columns can contain categorical or numerical values. They must also include labels.

You can follow this [link](https://docs.qiime2.org/2018.8/tutorials/metadata/) to learn more about metadata in QIIME2

Even if we are not doing it during this session, it is good practice to perform a validation of the mapping file.
To do that you can use Keemei, which is a  Google Sheets add-on for validating tabular bioinformatics file formats.
Instructions for installation and usage can be found [here](https://keemei.qiime2.org/)

### 1. Import data- Create an *artifact*

&#x1F536; To start working in QIIME2 we need to activate the QIIME environment. To do so first open a terminal:

![terminal](https://github.com/mbcarbonetto/qiime2-session/blob/master/annexed_files/terminal_button.png)

&#x1F536; Then type:

    source activate qiime2-2018.8

&#x1F536; We are now creating a working directory where we are going to place the output files of this session:

    mkdir working_dir

&#x1F536; You can also move the [input files](https://github.com/mbcarbonetto/qiime2-session/blob/master/hands_on_files.zip) you heve already dowloaded to this folder. You then move into the working directory.

    mv ~/Downloads/hands_on_files/ working_dir/
    cd working_dir

All data that is used as input to QIIME2 should be in form of QIIME2 artifacts, which contain information about the type of data and the format of the data. So, the first thing we need to do is import the sequence data files into a QIIME2 artifact.
The semantic type of our data is **SampleData[SequencesWithQuality]**, i.e *Sequences with quality scores (.fastq files), where each set of sequences is associated with a sample identifier (i.e. demultiplexed sequences, each file is a sample).*
In order to create an artifact we need a *manifest* file. This file will indicate the sample ID, the location of each fastq file and the direction of the reads, in our case just forward reads.

&#x1F536; Please complete the file with the correct path for each fastq file in your computer (replace "completePATH" for the real path). **Hint: the complete path starts with /home/**

&#x1F536; We are now ready to create a QIIME2 artifact with our data:

    qiime tools import \
    --type 'SampleData[SequencesWithQuality]' \
    --input-path hands_on_files/manifest.txt \
    --output-path single-end-demux.qza \
    --input-format SingleEndFastqManifestPhred33

With this command we are telling QIIME2 which **semantic type** (--type) to follow and what **format** (--input-format) of data we are importing. We are algo giving the path to the **input** fastq files with the manifest file (--input-path). Finally we are telling QIIME2 where to create the **output** file (--output-path) which will be called **single-end-demux.qza**

There are many other types of data that can be imported to QIIME2, each will have a different semantic type asigned and different protocols will be used to import them. Please follow [this link](https://docs.qiime2.org/2018.8/semantic-types/) for more information on semantic types in QIIME2 and [this one](https://docs.qiime2.org/2018.8/tutorials/importing/) to learn how to import different types of data.

Note: QIIME2 has a different argument structure than the previous QIIME version. We need to write first the command **qiime**, then the **plugin** name, then the **method** name and then the **arguments** (input, output and parameters). So, we need to follow this structure (the order of arguments is not important):

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

*Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/reads_QC/index.html) to take a look at the results.

This visualization allows to explore descriptive statistics of the sample sizes (i.e. min, max, median, mean, histogram) and samples quality based on Quality Score per base.

For our data set, the **Overvirew** information is not very informative since we know beforehand that all samples have the same size (10,000 reads). However, take a look at que **Interactive Quality Plot**:

:question: what is the read size when quality falls below Q20? (Take a look at the value in the 25th percentile and take note of this number)

<details><summary><b>Answer</b></summary> 

Quality falls below Q20 at position 243.

</details>


### 2.Quality filtering, denoising and feature picking using DADA2

We are now ready to perform quality control and feature picking. We are going to use DADA2. 
This method is based on correcting (where possible) Illumina sequencing mistakes. Features (also called amplicon sequence variants or ASV)  are inferred by a de novo process in which biological (true) sequences are discriminated from errors on the basis of the expectation that biological sequences are more likely to be repeatedly observed than are error-containing sequences.
You can get further details on the method [here](https://www.nature.com/articles/nmeth.3869) and [here](https://www.biorxiv.org/content/early/2015/08/06/024034).

&#x1F536; Run the following command in order to apply DADA2 on **single-end-demux.qza**:

    qiime dada2 denoise-single \
    --i-demultiplexed-seqs single-end-demux.qza \
    --p-trunc-len 243 \
    --p-trunc-q 0 \
    --output-dir DADA2

This command may take up to 30 minutes to run (in the computers set up for the session, in your own computer this will depend on the processor and RAM features)

&#x1F536; While you wait open a new terminal, open the QIIME2 environment and read the specifiactions of the parameters we have just used:

    source activate qiime2-2018.8
    qiime dada2 denoise-single --help
    
:question:  - Which criteria are we using to filter data by quality? Why?

<details><summary><b>Answer</b></summary>

We are using the --p-trunc-len parameter and decided to trim sequences at 243 bp because we can see in the *Interactive Quality Plot* that this is the position where Phred score falls below Q20. We could have choosen to trim reads based on -p-trunc-q 20. HOwever, this would have delivered a dataset with reads with length variation and DADA2 does not handle this correctly, so a trimming using the --p-trunc-len parameter is prefered.
    
 </details>
 
 
:question: - Are we remmoving chimeras? If so, which method are we using?

<details><summary><b>Answer</b></summary>

Yes we are by using the --p-chimera-method parameter set up by default. The default option is chimera removal by the "consensus" method where Chimeras are detected in samples individually, and sequences found chimeric in a sufficient fraction of samples are removed.

</details>


:question: - What are you expecting as output files?

<details><summary><b>Answer</b></summary>

You will get 3 output files: a "feature table"  artifact which is the resulting feature table with the information of feature (i.e. ASV) counts; a "representative sequence" artifact with the sequence information for each ASV and a "denoising stats" artifact with information on the denoising results.

</details>


&#x1F536; You can now crate a summary of the results and visualize it using *qiime tools view**

    qiime feature-table summarize \
    --i-table DADA2/table.qza \
    --o-visualization DADA2/table-dada2.qzv \
    --m-sample-metadata-file hands_on_files/mapping_file.tsv
    
    qiime tools view DADA2/table-dada2.qzv

*Alternative:* If you have not performed these steps just click [here](http://mbcarbonetto.github.io/qiime2_sessions/DADA2_table/index.html) to take a look at the results.

:question: Explore results:

- How many features has DADA2 resolved?

<details><summary><b>Answer</b></summary>
    
DADA2 detected 516 ASV

</details>


- How many reads remain after quality filtering?

<details><summary><b>Answer</b></summary>

There are 70,106 reads left after quality filtering.

</details>


- Which sample has the minimum frequency of reads? and the maximum?

<details><summary><b>Answer</b></summary>
    
The sample with the maximun number of reads is WT.unt.3 with 8,104 reads
The sample with the minimum number of reads is WT.day3.15 with 6,112 reads

</details>


We have now a feature table: **table.qza**, and we have assigned a representative sequence for each feature: **representative_sequences.qza**.
Before we continue with the analysis we can export the feature table so we can explore data with other software if needed.

&#x1F536; In order to export **table.qza** run:

    qiime tools export DADA2/table.qza --output-dir exported_table
    cd exported_table
    biom convert -i feature-table.biom -o feature-table.txt --to-tsv
    cd ..
    
We have just converted the feature table **table.qza** into **feature-table.biom**, which is useful for analysis with popular tools such as [Picrust2](https://github.com/picrust/picrust2/wiki) or [Phyloseq](https://joey711.github.io/phyloseq/) and **feature-table.txt** which can be visualized in any spreadsheet app.

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
      --p-sampling-depth 6112 \
      --m-metadata-file hands_on_files/mapping_file.tsv \
      --output-dir core-metrics-results

:question: Can you tell why **--p-sampling-depth** was set to **6112**?

<details><summary><b>Answer</b></summary>
    
This value was chosen based on the number of sequences in the **WT.day3.15** which is the one sample that has *fewer sequences*. Because most diversity metrics are sensitive to different sampling depths across different samples,we need to normalize data based on some criterium. Sub-sampling without replacement up to minimum sample size is one way to deal with this. This is probably not the best solution, but the most frequently used and accepted so far.

This script will randomly subsample the counts from each sample to the value provided for this parameter. For example, if you provide --p-sampling-depth 500, this step will subsample the counts in each sample without replacement so that each sample in the resulting table has a total count of 500. If the total count for any sample(s) are smaller than this value, those samples will be dropped from the diversity analysis. Choosing this value is tricky. You can make this choice by reviewing the information presented in the **table.qzv** file that was created above and choosing a value that is as high as possible (so you retain more sequences per sample) while excluding as few samples as possible.

</details>


Several visualization files were created, you will have one **.qzv** file for each beta diveristy metric that was calculated. 

&#x1F536; You can explore results using **qiime tools view**

    qiime tools view core-metrics-results/unweighted_unifrac_emperor.qzv

*Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/unweighted_unifrac_pcoa/data/index.html) to take a look at the visualization for unweighted Unifrac distance matrix.

&#x1F536; Take a look at the results for all beta diveristy distances. (Take advantage of the interactive functionalities)

:question: Can you observe any pattern or grouping of samples?

<details><summary><b>Answer</b></summary>

You can distinguish two groups: streptomicyn treated and untreated samples.

</details>


&#x1F536; We can now test for the significance of these grouping by using the following command:

    qiime diversity beta-group-significance \
    --i-distance-matrix core-metrics-results/unweighted_unifrac_distance_matrix.qza \
    --m-metadata-file hands_on_files/mapping_file.tsv \
    --m-metadata-column AntibioticUsage \
    --o-visualization core-metrics-results/unweighted_unifrac_AntibioticUsage-significance.qzv
    
Please take a look at the visualization files:

    qiime tools view core-metrics-results/unweighted_unifrac_AntibioticUsage-significance.qzv
     
*Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/unweighted_unifrac_sig/data/index.html) to take a look at the visualization.

:question: Does the test confirm what we have observed in the PCoA plot?

<details><summary><b>Answer</b></summary>

Yes it does for a p-value<0.05

</details>


Remember you can do the same analysis for every distance matrix, just change the **--i-distance-matrix** paramter and the name of the output file.

Note: **PERMANOVA** tests whether the distances between samples within the same group are more similar to each other than distances to samples on other group. The null hypothesis tested by PERMANOVA is that, under the assumption of exchangeability of the sample units among the groups, H0: “the centroids of the groups, as defined in the space of the chosen resemblance measure, are equivalent for all groups.” Thus, if H0 were true, any observed differences among the centroids in a given set of data will be similar in size to what would be obtained under random allocation of individual sample units to the groups (i.e., under permutation).
It is possible to choose **ANOSIM** method for testing group significance. ANOSIM is a modified version of the Mantel Test based on a standardized rank correlation between two distance matrices. The null hypothesis for the ANOSIM test is closely related to this, namely H0: “the average of the ranks of within-group distances is greater than or equal to the average of the ranks of between-group distances,” where a single ranking has been done across all inter-point distances in the distance matrix and the smallest distance (highest similarity) has a rank value of 1.
For further reading clik [here](https://esajournals.onlinelibrary.wiley.com/doi/full/10.1890/12-2010.1)

We have not yet taken a look at alpha diversity results.

&#x1F536; A good way to explore this is by making comparisons between groups of samples:

    qiime diversity alpha-group-significance \
    --i-alpha-diversity core-metrics-results/observed_otus_vector.qza \
    --m-metadata-file hands_on_files/mapping_file.tsv \
    --o-visualization core-metrics-results/observed_otus_vector-group-significance.qzv

    qiime tools view core-metrics-results/observed_otus_vector-group-significance.qzv
   
*Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/observed_otus/data/index.html) to take a look at the visualization.

:question: Is richnnes different between *Streptomycin treated* and *untreated* samples?

<details><summary><b>Answer</b></summary>

Yes, richness is higher in untreated mice.

</details>


Rememeber you can do the same analysis for every alpha diveristy metric calculated just change the **--i-alpha-diversity** parameter.

Note: **qiime diversity alpha-group-significance** uses Kruskal-Wallis nonparametric test. It also reports results of pairwise comparisons, in this case we only have two groups, so pairwise comparisons are not needed.

#### 5- Assign taxonomy to features with trained classifiers.

We are now ready to follow next step and classify the respesnetative sequences of each feature. In order to to this, we are going to use a pre-trained classifier. In this it has been pre- trained with Greengenes 13_8 99% OTUs full-16S rRNA gene length sequences. This is the Greengenes database (last release *13_18*) aligned at 99% similarity.

You will need to download **gg-13-8-99-nb-classifier.qza** from this [link](https://data.qiime2.org/2018.8/common/gg-13-8-99-nb-classifier.qza). Please copy the file to *hands_on_files* folder.

QIIME2 developers also provide some other pre-trained classifiers based on Silva and other databases [here](https://docs.qiime2.org/2018.8/data-resources/)

Taxonomic  classifiers perform best when they are trained based on your specific sample preparation and sequencing parameters, including the primers that were used for amplification and the length of your sequence reads. In this case, we are using a database based on the full-length 16S rRNA gene, but a customized database including just V3-V4 regions would probably be a better choice. We are not customizinmg the database or training the classifier in this tutorial, but you can find a detailed tutorial [here](https://docs.qiime2.org/2018.4/tutorials/feature-classifier/).

&#x1F536; We are ready now to run the classification:

    qiime feature-classifier classify-sklearn \
    --i-classifier hands_on_files/gg-13-8-99-nb-classifier.qza \
    --i-reads DADA2/representative_sequences.qza \
    --o-classification taxonomy.qza

&#x1F536; We need now to create visualization files to explore the results:
 
     qiime metadata tabulate \
     --m-input-file taxonomy.qza \
     --o-visualization taxonomy.qzv
     
     qiime tools view taxonomy.qzv
     
 This will output a table with each feature ID; its classification and the confidence level for the taxonomy assignment. Note that you can export a .tsv with these results. 
 
 *Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/taxonomy_list/data/index.html) to take a look at the visualization.
 
&#x1F536; You can also create a visualization file based on bar plots using the following command:

    qiime taxa barplot \
    --i-table DADA2/table.qza \
    --i-taxonomy taxonomy.qza \
    --m-metadata-file hands_on_files/mapping_file.tsv \
    --o-visualization taxa-bar-plots.qzv
    
    qiime tools view taxa-bar-plots.qzv

*Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/taxa_bar_plots/data/index.html) to take a look at the visualization.
 
You can now explore results, take into account:

- Level 1 = Kingdom (e.g Bacteria)
- Level 2 = Phylum (e.g Actinobacteria)
- Level 3 = Class (e.g Actinobacteria)
- Level 4 = Order (e.g Actinomycetales)
- Level 5 = Family (e.g Streptomycetaceae)
- Level 6 = Genus (e.g Streptomyces)
- Level 7 = Species (e.g mirabilis)

Note that you can sort samples by metadata and change colors and taxonomic levels. Morevoer, you can download figures and data in .csv format.

#### 6- Test for differential abundances between groups of samples using ANCOM

We can now explore if the differences observed in the plots for treated and untreated mice are statisticaly significant.
In this case we will use [ANCOM](https://www.ncbi.nlm.nih.gov/pubmed/26028277) (Analysis of Composition of Microbiomes).
This method accounts for the compositional nature of 16S data. In brief ANCOM adds a pseudocount to each count value and then compares the log ratio of the abundance of each taxon to the abundance of all the remaining  taxa one at a time. (Transformations  after [Aitchison 1986](https://scholar.google.com/scholar_lookup?author=J.+Aitchison+&publication_year=1986&title=The+Statistical+Analysis+of+Compositional+Data)).
If you want to get a deeper insight into the problem of compositional data please read [this](https://www.frontiersin.org/articles/10.3389/fmicb.2017.02224/full#B2). if you want furhter detais on how ANCOM works please read [this](http://mortonjt.blogspot.pt/2016/06/ancom-explained.html).

&#x1F536; We will test for differential abundances at the genus level. In order to do that, we need first to get the table of relative abundances of reads for each genera for each sample:

    qiime taxa collapse \
    --i-table DADA2/table.qza \
    --i-taxonomy taxonomy.qza \
    --p-level 6 \
    --o-collapsed-table table-dada2-l6.qza
    
&#x1F536; ANCOM operates on table based on frequencies of features/taxa on a per-sample basis, but cannot tolerate frequencies of zero. To build this table or **composition artifact**we ned to add a pseudocount.

    qiime composition add-pseudocount \
    --i-table table-dada2-l6.qza \
    --o-composition-table comp-table-dada2-l6.qza
    
&#x1F536; We are now ready to run the differential abundance test:

    qiime composition ancom \
    --i-table comp-table-dada2-l6.qza \
    --m-metadata-file hands_on_files/mapping_file.tsv \
    --m-metadata-column AntibioticUsage \
    --o-visualization l6-ancom-AntibioticUsage.qzv
    
    qiime tools view l6-ancom-AntibioticUsage.qzv
 
 *Alternative:* If you have not performed this step just click [here](https://mbcarbonetto.github.io/qiime2_sessions/ANCOM_genus/data/index.html) to take a look at the visualization.
      
&#x1F536; Let's take a look at the results:
    
:question: Which genus/genera differ in abundance between treated and untreated samples? In which group is each genus more abundant?

<details><summary><b>Answer</b></summary>

Bacteroides and an unamed genus in Enterobacteriaceae family are more abundant in streptomicyn treated mice. While an unnamed genus in Clostridiales group is more abundant in untreated mice.

</details>


**Note:** ANCOM does not report p-values but a table with information on the rejection (or not) of H0. They also provide the statistic "W" values and information of the distribution of data in percentiles for each tested group.
