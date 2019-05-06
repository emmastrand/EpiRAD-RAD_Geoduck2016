# Background Information
Author: Emma Strand  
Last edited: 20190422

##dDocent 

##EpiRADseq




##RADseq

## Data File Formats
###FASTA files

###FASTQ files

###SAM/BAM format

###VCF format 

## Multiqc Results

#### General Statistics Table


## Read Mapping

Read mapping is the process of aligning reads from the data set to a reference genome. Image: [Galaxy Project](https://galaxyproject.github.io/training-material/topics/sequence-analysis/tutorials/mapping/tutorial.html).

![Read mapping](https://galaxyproject.github.io/training-material/topics/sequence-analysis/images/mapping/mapping.png)

Visualization of matches, mismatches, and gap penalties. Matches are caused by conservation of the seqeuence, compared to mistmatches are caused by a subsitution in the sequence. Gaps are caused by insertions or deletions (sometimes referred to as indels) and are represented by a dash mark. Image: [SeqAn](https://seqan.readthedocs.io/en/master/Tutorial/DataStructures/Alignment/ScoringSchemes.html). 

![Match mismatch gap](https://seqan.readthedocs.io/en/master/_images/alig_ins_del.png)

## Single Nucleotide Polymorphisms (SNPs)

FreeBayes is a program used to identify SNPs between the samples and reference genome based on haplotype differences. Image: [HBC Training](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionVI/lessons/02_variant-calling.html).

![FreeBayes](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionVI/img/freebayes_2.png)