---
title: "Microbial diversity analysis using 16S rRNA gene- Trainning session with QIIME2"
author: "Belen Carbonetto"
date: "March 26, 2018"
output: html_document
---
## Session overview

This session will introduce state-of-the art analysis of 16s rRNA amplicons to study microbial diversity. Participants will learn how to use QIIME2 analysis package to asses microbial community diversity and composition.

## Target audience

PhD students and post-doctoral researchers who are planning to use Illumina based 16S rRNA gene amplicon sequencing to study microbial diversity.

## Learning objectives

-Learn what diversity is; understand concepts of alpha (richness and evenness) and beta diversity.

-Perform pre-process of sequences: remove primers and barcodes, demultiplex samples.

-Do quality filtering, denoising and picking of features and representative sequences with DADA2 and Deblur

-Assign taxonomy to features with trained classifiers.

-Align sequences and infer phylogeny.

-Calculate alpha and beta diversity.

-Test for differential abundances between groups of samples using  ANCOM and gneiss.

## Learning outcomes

###**LO1- Understand core concepts in diversity analysis**###
 
 LO1.1- Explain what alpha diversity is. What are the components of alpha diversity?
 
 LO1.2- What information can you take from a rarefaction curve?
 
 LO1.3- How can you measure beta diversity? What are the differences between distances?
 
 LO1.4- List advantages of 16S rRNA subunit gene as marker for diversity analysis.
 
###**LO2- List steps in 16S rRNA amplicon based microbial diveristy analysis**###

###**LO3- Prepare a mapping file**###

###**LO4- Perform demultiplexing of reads**###

 LO4.1- List the options for demultiplexing reads
 LO4.2- Explore quality report and plot and decide on the quality filters to apply next
 
###**LO5- Perform quality control and cluster reads into features**###

 LO5.1- Perform quality filtering by phred score and explore results
 LO5.2- Name the main differences between subOTUs and OTU tables
 LO5.3- Perform feature table construction and finish quality control
 LO5.4- Create summaries
 
###**LO6- Perform alignment and a phylogenetic tree**##

 LO6.1- Perform a multiple sequence alignment of the sequences
 LO6.2- Mask the alignment
 LO6.3- Create a phylogenetic tree
 
###**LO7- Perform diveristy analysis**###

 LO7.1- List the options of alpha and beta diversity metrics that can be measured
 LO7.2- Decide on the subsampling depth to use for diversity calculations. How many samples are excluded?
 LO7.3- Run diverstiy analysis and explore results. 
          Can you observe any community structure defined by metadata categories?
