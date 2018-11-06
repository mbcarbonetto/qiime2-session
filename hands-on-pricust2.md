---
title: "Hands-on: Prediction of functional diversity using 16S rRNA gene data with PICRUSt2 (*q2 picrust2*)"
author: "Belen Carbonetto"
date: "November 2018"
Version 3
output: html_document
---

## Prediction of functional diveristy using DADA2 output files and PICRUSt2 plugin in QIIME2

### Learning outcomes:

**During this hands-on session you will learn how to:**

Use DADA2 output to calculate [EC](https://bitesizebio.com/10683/understand-ec-numbers-in-5-minutes-part-i-how-ec-numbers-work/) numbers and [KO](https://www.genome.jp/kegg/ko.html) predictions. 

MetaCyc pathway coverages and abundances will also be calculated and *.tsv* files with prediction results will be exported.

**Notes:** [PICRUSt2](https://github.com/picrust/picrust2/wiki) and the plugin [*q2 picrust2*](https://github.com/picrust/picrust2/wiki/q2-picrust2-Tutorial) have already been installed in your working station in order to follow this tutorial.
If you are following the tutorial on your own please find instructions for the installation [here](https://library.qiime2.org/plugins/q2-picrust2/13/)

This tutorial is ment to be followed after the QIIME2 [hands-on](https://github.com/mbcarbonetto/qiime2-session/blob/master/Hands-on-V3.md) session.

- Every task that you will perfom is marked with &#x1F536;

### 0. Data set and input files

&#x1F536; You will need to download the reference files we will use in this session from this [link](https://github.com/mbcarbonetto/qiime2-session/tree/master/picrust). Please download both files: *reference.fna.qza* and *reference.tre.qza*.

The input files will be the output files from **qiime dada2 denoise-single** command: the "feature table" artifact and the "representative sequence" artifact (i.e. table.qza and representative_sequences.qza in the QIIME2 [hands-on](https://github.com/mbcarbonetto/qiime2-session/blob/master/Hands-on-V3.md) sessions)

### 1. Place the ASVs against the PICRUSt2 reference multiple-sequence alignment and phylogeny.

&#x1F536; The input files for this step are the representive sequences (i.e. the ASVs) and the reference tree and alignment.

**Hint:** It may be helpful to create a working directory for this step and copy all input files there, alternatively you can call the complete path for each file. Please note the paths in this tutorial may differ from your paths depending where you placed the refences and/or if you are using IIME2-sessions hands-on output files.

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

&#x1F536; Now that ASVs are placed on the reference phylogeny we can proceed to predict metegenomes and to calculate pathway abundances. (PICRUSt2 developers estimate that this setp takes ~40 min and 5GB of RAM - this will be much faster if you can set more threads).

    qiime picrust2 custom-tree-pipeline \
    --i-table DADA2/table.qza \
    --i-tree picrust2/results/tree.qza \
    --output-dir picrust2/results/q2-picrust2_output \
    --p-threads 2 \
    --p-hsp-method mp \
    --p-max-nsti 2
 
Note: the *--p-hsp-method= pic* phylogenetic independent contrast hidden-state prediction is fastest. However PICRUSt2 developers suugest to use the *mp method* maximum parsimony. 
Available methods are: maximum parsimony (mp), empirical probabilities (emp_prob), subtree averaging (subtree_average), phylogenetic independent contrast (pic), or squared-change parsimony (scq).

The --p-max-nsti option specifies how distantly placed a sequence needs to be in the reference phylogeny before it is excluded. The default cut-off is 2. In human datasets used for testing PICRUSt2 the only ASVs above this default cut-off were 18S sequences erroneously in 16S datasets, which suggests this cut-off is highly lenient. For environmental datasets a higher proportion of ASVs may be thrown out based on this default cut-off (Douglas et al. In Prep).

**Hint:** for further details on parameters options allways type **--help** at the end on the command:

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
    
Follow this [link](https://github.com/picrust/picrust2/wiki) for citation information and more details on PICRUSt2 pipeline


