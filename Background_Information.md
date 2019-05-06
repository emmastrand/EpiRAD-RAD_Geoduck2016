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
Reading a Multiqc report: [Using MultiQC Reports Video](https://www.youtube.com/watch?v=qPbIlO_KWN0).

#### General Statistics Table
The key statistical data presented in this table will be variable depending on what kind of data is being analyzed and what programs were used. See [Multiqc](https://multiqc.info/). Samples will be aligned on the left to be easily comparable. 

![General_stats](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/_images/General_stats.png?raw=true)

Options shown above:  
% Dups = Percentage of duplicate reads.   
% GC = Average percentage of GC content  
M Seqs = Total number of sequences (millions)

Additional options:  
Length = Average sequence length (basepairs)  
% mCpG = percentage of cytosines methylated in CpG content (using Bismark)  
M C's = Total number of C's analyzed in millions (using Bismark)  
% Aligned = percentage aligned   
% Trimmed = percentage of total base pairs trimmed (using CutAdapt)

#### 

## Read Mapping

Read mapping is the process of aligning reads from the data set to a reference genome. Image: [Galaxy Project](https://galaxyproject.github.io/training-material/topics/sequence-analysis/tutorials/mapping/tutorial.html).

![Read mapping](https://galaxyproject.github.io/training-material/topics/sequence-analysis/images/mapping/mapping.png)

Visualization of matches, mismatches, and gap penalties. Matches are caused by conservation of the seqeuence, compared to mistmatches are caused by a subsitution in the sequence. Gaps are caused by insertions or deletions (sometimes referred to as indels) and are represented by a dash mark. Image: [SeqAn](https://seqan.readthedocs.io/en/master/Tutorial/DataStructures/Alignment/ScoringSchemes.html). 

![Match mismatch gap](https://seqan.readthedocs.io/en/master/_images/alig_ins_del.png)

## Single Nucleotide Polymorphisms (SNPs)

FreeBayes is a program used to identify SNPs between the samples and reference genome based on haplotype differences. Image: [HBC Training](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionVI/lessons/02_variant-calling.html).

![FreeBayes](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionVI/img/freebayes_2.png)