---
title: "Hands-on: Prediction of functional diveristy using 16S rRNA gene data with PICRUSt2 (*q2 picrust2*)"
author: "Belen Carbonetto"
date: "November 2018"
Version 3
output: html_document
---

## Prediction of functional diveristy using DADA2 output files and PICRUSt2 plugin in QIIME2

### Learning outcomes:

**During this hands-on session you will learn how to:**

Use DADA2 output to infer [EC](https://bitesizebio.com/10683/understand-ec-numbers-in-5-minutes-part-i-how-ec-numbers-work/) numbers and [KO](https://www.genome.jp/kegg/ko.html) predictions. 

MetaCyc pathway coverages and abundances will also be calculated and .tsv files with results will be exported.

**Notes:** Picrust2 and the plugin *q2 picrust2* has already been installed in your working station in order to follow this tutorial.
If you are following the tutorial on your own please find instructions for the installation [here](https://library.qiime2.org/plugins/q2-picrust2/13/)

This tutorial is ment to be followed after the QIIME2-sessions [hands-on](https://github.com/mbcarbonetto/qiime2-session/blob/master/Hands-on-V3.md)

- Every task that you will perfom is marked with &#x1F536;

- Questions to further analyse results will be marked with :question:

### 0. Data set and input files

&#x1F536; You will need to download the reference files we will use in this session from this [link](https://github.com/mbcarbonetto/qiime2-session/tree/master/picrust). Please download both files:*reference.fna.qza* and * 	reference.tre.qza*.

The input files will be the output files from **qiime dada2 denoise-single**:the "feature table" artifact and the "representative sequence" artifact (i.e. table.qza and representative_sequences.qza)

### 1. Place the ASVs against the PICRUSt2 reference multiple-sequence alignment and phylogeny.

&#x1F536; The input files of this step are the representive sequences (i.e. the ASVs) and the refence tree and alignment.

**Hint:** It may be helpful to create a working directory for this step and copy all input files there, alternatively you can call the complete path for each file. Please note the paths maybe different to the paths is this tutorial

    mkdir picrust2
    mv Donwloads/reference.fna.qza picrust2/
    mv Donwloads/reference.tre.qza picrust2/
    
    qiime fragment-insertion sepp \
    --i-representative-sequences DADA2/representative_sequences.qza \
    --p-threads 1 \
    --i-reference-alignment picrust2/reference.fna.qza \
    --i-reference-phylogeny picrust2/reference.tre.qza \
    --output-dir picrust2/results
 
### 2. Run the full PICRUSt2 pipeline.

&#x1F536; Now that ASVs are placed on the reference phylogeny we can proceed to the predict metegenomes and infer pathway abundances. (PICRUSt2 developers estimate that this setp takes ~40 min and 5GB of RAM - this will be much faster if you can set more threads).

    qiime picrust2 custom-tree-pipeline \
    --i-table DADA2/table.qza \
    --i-tree picrust2/results/tree.qza \
    --output-dir picrust2/results/q2-picrust2_output \
    --p-threads 2 \
    --p-hsp-method mp \
    --p-max-nsti 0.15
 
Note: the *--p-hsp-method= pic* phylogenetic independent contrast hidden-state prediction is fastest. However PICRUSt2 developers suugest to use the *mp method* maximum parsimony. 
Available methods are: maximum parsimony (mp), empirical probabilities (emp_prob), subtree averaging (subtree_average), phylogenetic independent contrast (pic), or squared-change parsimony (scq).

**Hint:** for further detais on parameters options allways type **--help** at the end on the command:

    qiime picrust2 custom-tree-pipeline --help
   
 
### 3. Create visualization files.
&#x1F536;

    qiime feature-table summarize \
    --i-table picrust2/results/q2-picrust2_output/pathway_abundance.qza \
    --o-visualization picrust2/results/q2-picrust2_output/pathway_abundance.qzv
    
    qiime feature-table summarize \
    --i-table picrust2/results/q2-picrust2_output/pathway_coverage.qza \
    --o-visualization picrust2/results/q2-picrust2_output/pathway_coverage.qzv

    qiime feature-table summarize \
    --i-table picrust2/results/q2-picrust2_output/ko_metagenome.qza \
    --o-visualization picrust2/results/q2-picrust2_output/ko_metagenome.qzv

    qiime feature-table summarize \
    --i-table picrust2/results/q2-picrust2_output/ec_metagenome.qza \
    --o-visualization picrust2/results/q2-picrust2_output/ec_metagenome.qzv
    
 &#x1F536; To visualize results use the foloowing command:
 
    qiime tools view picrust2/results/q2-picrust2_output/pathway_abundance.qzv
 
You can use the same command for each .qzv file
 
### 4. Extract tables

&#x1F536; Follow this commands in order to get .tsv with the results to use as input in other softwate (spreadsheets, R, etc)
    
    qiime tools export --input-path picrust2/results/q2-picrust2_output/pathway_abundance.qza --output-path picrust2/results/q2-picrust2_output/exported_pathway_abn
    qiime tools export --input-path picrust2/results/q2-picrust2_output/pathway_coverage.qza --output-path picrust2/results/q2-picrust2_output/exported_pathway_cov
    qiime tools export --input-path picrust2/results/q2-picrust2_output/ec_metagenome.qza --output-path picrust2/results/q2-picrust2_output/exported_EC
    qiime tools export --input-path picrust2/results/q2-picrust2_output/ok_metagenome.qza --output-path picrust2/results/q2-picrust2_output/exported_KO
    biom convert -i picrust2/results/q2-picrust2_output/exported_pathway_abn/feature-table.biom -o  picrust2/results/q2-picrust2_output/exported_pathway_abn/feature-table.tsv --to-tsv
    biom convert -i picrust2/results/q2-picrust2_output/exported_pathway_cov/feature-table.biom -o  picrust2/results/q2-picrust2_output/exported_pathway_cov/feature-table.tsv --to-tsv
    biom convert -i picrust2/results/q2-picrust2_output/exported_EC/feature-table.biom -o  picrust2/results/q2-picrust2_output/exported_EC/feature-table.tsv --to-tsv
    biom convert -i picrust2/results/q2-picrust2_output/exported_KO/feature-table.biom -o  picrust2/results/q2-picrust2_output/exported_KO/feature-table.tsv --to-tsv
    
Follow this [link](https://github.com/picrust/picrust2/wiki) for citation iformation and more details on PICRUSt2 pipeline


