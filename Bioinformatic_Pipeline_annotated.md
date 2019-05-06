# *P. generosa* RAD and EpiRADseq Bioinformatic Pipeline Anlaysis

Author: Emma Strand  
Last Edited: 20190422

Data upload and analyzed on KITT. User logged in before following steps are completed. 
## Using Terminal
### 1. Set-Up (downloading software programs, creating folders, and uploading raw data)

**Downloading Bioconda (skip this step if Bioconda has already been downloaded).**

```
# downloading Miniconda software
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
$ chmod +x Miniconda3-latest-Linux-x86_64.sh
$ ./Miniconda3-latest-Linux-x86_64.sh

# restarting window with source command
$ source ~/.bashrc

# adding different channels
$ conda config --add channels defaults
	# should get: "warning: 'defaults' already in 'channels' list, moving to the top"
$ conda config --add channels bioconda
$ conda config --add channels conda-forge

# to see if this worked with cat command
$ cat .condarc
```

> Bioconda is a bioinformatics software package manager. See more at [https://bioconda.github.io](https:///bioconda.github.io).

**Create and activate an environment in conda**  

```
$ conda create -n Geoduck_EpiRAD
$ conda activate Geoduck_EpiRAD

# the beginning of the line should then look like: (Geoduck_EpiRAD) [estrand@KITT ~]$
```

**Creating and entering a directory for this project** 

```
$ mkdir Final_Project # creates a directory
$ cd Final_Project # navigates into this new directory

# the beginning of the line should now look like: (Geoduck_EpiRAD) [estrand@KITT Final_Project]$
```

**Copying raw data files to directory on KITT**

```
# See Raw_Data folder on EpiRAD-RAD_Geoduck2016 repository for contents (multiplexed files, barcode files, and checksums.md5 file). 

$ mkdir Raw_Data
$ cd Raw_Data
$ wget (link to raw data file) 

# to check the format of a file 
$ file JD002A_S131_L005_R1_001.fastq.gz
output: 
JD002A_S131_L005_R1_001.fastq.gz: gzip compressed data, extra field, max compression

```

### 2. Checking Quality of Raw Data

**Double check the files downloaded correctly using checksum**

```
$ wget (link to checksums.md5 file)
$ md5sum -c checksums.md5

output:
JD002A_S131_L005_R1_001.fastq.gz: OK
JD002A_S131_L005_R2_001.fastq.gz: OK
JD002D_S134_L005_R1_001.fastq.gz: OK
JD002D_S134_L005_R2_001.fastq.gz: OK
JD002G_S137_L005_R1_001.fastq.gz: OK
JD002G_S137_L005_R2_001.fastq.gz: OK
JD002H_S138_L005_R1_001.fastq.gz: OK
JD002H_S138_L005_R2_001.fastq.gz: OK
JD002I_S139_L005_R1_001.fastq.gz: OK
JD002I_S139_L005_R2_001.fastq.gz: OK
JD002K_S141_L005_R1_001.fastq.gz: OK
JD002K_S141_L005_R2_001.fastq.gz: OK
JD002L_S142_L005_R1_001.fastq.gz: OK
JD002L_S142_L005_R2_001.fastq.gz: OK
```
> Calculating a md5sum is a way of checking file integrity. In other words, confirming that two files contain the same information and that data wasn't lost in the downloading process.

**FastQC Analysis**  

```
$ mkdir multiplexed_fastqc_results
$ cd multiplexed_fastqc_results
$ conda install -c bioconda fastqc
$ fastqc ../*fastq.gz 
$ mv *fastqc.* multiplexed_fastqc_results/ # moves fastqc files to fastqc directory 
```
> [FastQC](https://dnacore.missouri.edu/PDF/FastQC_Manual.pdf) is a program designed to visualize the quality of high throughput sequencing datasets. The report will highlight any areas where the data looks unsual or unreliable.

**Multiqc Analysis**

```
$ conda install -c bioconda multiqc
$ multiqc .

# copy fastqc and multiqc files to directory outside of KITT
# run the following commands outside of KITT
$ scp -r -P XXXX x@kitt.uri.edu:/home/estrand/Final_Project/Raw_Data/multiplexed_fastqc_results/ /Users/emmastrand/MyProjects/EpiRAD-RAD_Geoduck2016
# XXXX above indicates the password specific for our lab's KITT.
# scp = secure copy, -r = indicates copying a folder/repository 

# open multiqc_report to view the quality of raw, multiplexed data files
$ mv multiqc_report.html multiplexed_multiqc_report.html # rename file

```
> Multiqc creates a single report from the FastQC results. See [Background_Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md) for a breakdown of Multiqc.


### 3. Demultiplexing Data Files

Each library contains multiple species and both EpiRADseq and ddRADseq reads for several individuals. From raw data files, R1 indicates forward and R2 indicates reverse sequences. 

> Demultiplexing refers to the step in processing where you use barcode information to identify which sequences came from what sample after multiple samples are sequenced together. Barcodes refer to the unqiue sequence that were added to each sample before all the samples are mixed together ([Happy Belly Bioinformatics](https://astrobiomike.github.io/amplicon/demultiplexing)).

#### Using the program [STACKS](http://catchenlab.life.illinois.edu/stacks/):  
The following commands are already installed on the system. See the above STACKS link for more information.  
See the [Raw_Data](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/tree/master/Raw_Data) folder on github for barcode file format. Each library needs a separate barcodes file in txt format.

```
# example of barcode file. The name and barcode need to be separated by a single tab. 
# The programs used in this pipeline (dDocent) need the name format to be Population_SampleID.

Epi_Geo4	TGCAT
dd_Geo4	AAGGA
Epi_Geo3	CAACC
dd_Geo3	TCGAT	
```

**Turning barcode files into a list of barcodes**

```
# make sure to be in Raw_Data directory 
# the -f2 command selects the second column of the file

$ cut -f2 LibraryA_barcodes.txt > barcodes_A
$ cut -f2 LibraryD_barcodes.txt > barcodes_D
$ cut -f2 LibraryG_barcodes.txt > barcodes_G
$ cut -f2 LibraryH_barcodes.txt > barcodes_H
$ cut -f2 LibraryI_barcodes.txt > barcodes_I
$ cut -f2 LibraryK_barcodes.txt > barcodes_K
$ cut -f2 LibraryL_barcodes.txt > barcodes_L

# To view the new barcodes file 
$ head barcodes_A
```

**Using the function radtags to demultiplex each file**

```
# download renaming script from J. Puritz 
$ curl -L -O https://github.com/jpuritz/dDocent/raw/master/Rename_for_dDocent.sh

## Library A
$ process_radtags -1 JD002A_S131_L005_R1_001.fastq.gz -2 JD002A_S131_L005_R2_001.fastq.gz -b barcodes_A -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 2 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002A_S131_L005_R1_001.fastq.gz]
  Reading data from:
  JD002A_S131_L005_R1_001.fastq.gz and
  JD002A_S131_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...
  40846816 total reads; -40537310 ambiguous barcodes; -92111 ambiguous RAD-Tags; +118779 recovered; -0 low quality reads; 217395 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

40846816 total sequences
40537310 ambiguous barcode drops (99.2%)
       0 low quality read drops (0.0%)
   92111 ambiguous RAD-Tag drops (0.2%)
  217395 retained reads (0.5%)
  
# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryA_barcodes.txt

# create directory for files that failed demultiplexing 
# These will include the other species we don't need in the library
# This has to be done after each step because the same barcode ID numbers were used in each library. If not done after each library, the previous files will be overrided
$ mkdir LibA_failed_demultiplexed
$ mv sample* LibA_failed_demultiplexed

## Library D
$ process_radtags -1 JD002D_S134_L005_R1_001.fastq.gz -2 JD002D_S134_L005_R2_001.fastq.gz -b barcodes_D -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 4 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002D_S134_L005_R1_001.fastq.gz]
  Reading data from:
  JD002D_S134_L005_R1_001.fastq.gz and
  JD002D_S134_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...21M...22M...23M...24M...25M...26M...
  52123616 total reads; -38694182 ambiguous barcodes; -662558 ambiguous RAD-Tags; +1644787 recovered; -0 low quality reads; 12766876 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

52123616 total sequences
38694182 ambiguous barcode drops (74.2%)
       0 low quality read drops (0.0%)
  662558 ambiguous RAD-Tag drops (1.3%)
12766876 retained reads (24.5%)

# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryD_barcodes.txt

# moving failed files
$ mkdir LibD_failed_demultiplexed
$ mv sample* LibD_failed_demultiplexed

## Library G
$ process_radtags -1 JD002G_S137_L005_R1_001.fastq.gz -2 JD002G_S137_L005_R2_001.fastq.gz -b barcodes_G -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 2 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002G_S137_L005_R1_001.fastq.gz]
  Reading data from:
  JD002G_S137_L005_R1_001.fastq.gz and
  JD002G_S137_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...
  40874870 total reads; -38654542 ambiguous barcodes; -148650 ambiguous RAD-Tags; +383863 recovered; -0 low quality reads; 2071678 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

40874870 total sequences
38654542 ambiguous barcode drops (94.6%)
       0 low quality read drops (0.0%)
  148650 ambiguous RAD-Tag drops (0.4%)
 2071678 retained reads (5.1%)

# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryG_barcodes.txt

## Library H
$ process_radtags -1 JD002H_S138_L005_R1_001.fastq.gz -2 JD002H_S138_L005_R2_001.fastq.gz -b barcodes_H -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 4 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002H_S138_L005_R1_001.fastq.gz]
  Reading data from:
  JD002H_S138_L005_R1_001.fastq.gz and
  JD002H_S138_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...21M...22M...23M...24M...25M...
  50610358 total reads; -48028544 ambiguous barcodes; -344758 ambiguous RAD-Tags; +531048 recovered; -0 low quality reads; 2237056 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

50610358 total sequences
48028544 ambiguous barcode drops (94.9%)
       0 low quality read drops (0.0%)
  344758 ambiguous RAD-Tag drops (0.7%)
 2237056 retained reads (4.4%)
 
# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryH_barcodes.txt

## Library I
$ process_radtags -1 JD002I_S139_L005_R1_001.fastq.gz -2 JD002I_S139_L005_R2_001.fastq.gz -b barcodes_I -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 2 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002I_S139_L005_R1_001.fastq.gz]
  Reading data from:
  JD002I_S139_L005_R1_001.fastq.gz and
  JD002I_S139_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...21M...22M...23M...
  46619460 total reads; -45667282 ambiguous barcodes; -170541 ambiguous RAD-Tags; +264165 recovered; -0 low quality reads; 781637 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

46619460 total sequences
45667282 ambiguous barcode drops (98.0%)
       0 low quality read drops (0.0%)
  170541 ambiguous RAD-Tag drops (0.4%)
  781637 retained reads (1.7%)

# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryI_barcodes.txt


## Library K
$ process_radtags -1 JD002K_S141_L005_R1_001.fastq.gz -2 JD002K_S141_L005_R2_001.fastq.gz -b barcodes_K -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 4 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002K_S141_L005_R1_001.fastq.gz]
  Reading data from:
  JD002K_S141_L005_R1_001.fastq.gz and
  JD002K_S141_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...21M...22M...23M...24M...25M...26M...
  53758236 total reads; -52808148 ambiguous barcodes; -249620 ambiguous RAD-Tags; +368108 recovered; -0 low quality reads; 700468 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

53758236 total sequences
52808148 ambiguous barcode drops (98.2%)
       0 low quality read drops (0.0%)
  249620 ambiguous RAD-Tag drops (0.5%)
  700468 retained reads (1.3%)
  
# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryK_barcodes.txt


## Library L
$ process_radtags -1 JD002L_S142_L005_R1_001.fastq.gz -2 JD002L_S142_L005_R2_001.fastq.gz -b barcodes_L -e PstI --renz_2 mspI -r -i gzfastq

output:
Processing paired-end data.
Using Phred+33 encoding for quality scores.
Found 1 paired input file(s).
Searching for single-end, inlined barcodes.
Loaded 2 barcodes (5bp).
Will attempt to recover barcodes with at most 1 mismatches.
Processing file 1 of 1 [JD002L_S142_L005_R1_001.fastq.gz]
  Reading data from:
  JD002L_S142_L005_R1_001.fastq.gz and
  JD002L_S142_L005_R2_001.fastq.gz
  Processing RAD-Tags...1M...2M...3M...4M...5M...6M...7M...8M...9M...10M...11M...12M...13M...14M...15M...16M...17M...18M...19M...20M...21M...22M...23M...24M...25M...26M...27M...
  54293144 total reads; -52865304 ambiguous barcodes; -125498 ambiguous RAD-Tags; +213529 recovered; -0 low quality reads; 1302342 retained reads.
Closing files, flushing buffers...
Outputing details to log: './process_radtags.log'

54293144 total sequences
52865304 ambiguous barcode drops (97.4%)
       0 low quality read drops (0.0%)
  125498 ambiguous RAD-Tag drops (0.2%)
 1302342 retained reads (2.4%)

# rename for dDocent
$ bash Rename_for_dDocent.sh LibraryL_barcodes.txt

```
`-e	 PstI` = which 5' restriction site was used.  
`--renz_2 mspI` = which 3' restriction site was used. Both mspI and HpaII at the same restriction site so either will work here.  
`-r` tells radtags to fix cut sites/barcodes that have up to 1-2 mutations in them.  
`-i gzfastq` = format of the sequence

**Create a new directory and move demultiplexed files** 

```
# Because STACKS is default is maximum 1 barcode mismatch, but it is unclear is 1 or 0 mismatch maximum is better for this data set. 
# Because I might have multiple demutliplexed data folders in the future, I will specify with the folder name now.
  
$ mkdir 1mis_Demultiplexed_Data
$ mv Epi* 1mis_Demultiplexed_Data
$ mv dd* 1mis_Demultiplexed_Data
$ ls 1mis_Demultiplexed_Data
```
There should be 40 total files: 10 individual geoducks with 2 EpiRAD files and 2 RAD files. Each EpiRAD and RAD for each geoduck should have a forward (F) and reverse (R) sequence.

**Counting raw reads for each file**

```
$ echo $(zcat *.fastq.gz|wc -l)/4|bc

output:

```
> Each read in a fastq file has 4 lines: 1. identifier 2. sequence 3. description that starts with "+" 4. quality for each base in the second line. Raw reads can be counted by calculating the number of lines in the file and dividing by 4. `echo	` is used to start an argument and writes the argument as a standard output.`zcat` is used for zipped files. 

**Checking the quality of data post-demultiplexing**

```
$ cd 1mis_Demultiplexed_Data
$ mkdir 1mis_fastqc_results
$ cd 1mis_fastqc_results
$ fastqc ../*fq.gz 
$ mv *fastqc.* 1mis_fastqc_results/ # moves fastqc files to fastqc directory

$ multiqc .

# copy fastqc and multiqc files to directory outside of KITT
# run the following commands outside of KITT
$ scp -r -P XXXX x@kitt.uri.edu:/home/estrand/Final_Project/Raw_Data/1mis_fastqc_results/ /Users/emmastrand/MyProjects/EpiRAD-RAD_Geoduck2016
``` 
*Link to mutliqc html*

### 4. Quality Filtering, *De Novo* Assembly, Read Mapping, SNP Calling, and SNP Filtering

**Installing and starting dDocent** 

```
$ conda install dDocent # you only need to do this once. skip if already downloaded.
$ dDocent

# make sure to be in 1mis_Demultiplexed_Data directory
# downloading reference genome 
$ wget http://owl.fish.washington.edu/halfshell/genomic-databank/Pgenerosa_v070.fa
```

> dDocent is a bioinformatics program created by [Dr. Jon Purtiz](https://github.com/jpuritz) that is specifically designed for different types of RAD sequencing. See the [Background Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md) file in this repository for more information.   
> dDocent requires a raw, demultiplexed file in gzipped fastq format for each individual.  
> Naming convention example: "Pop1_Sample1.F.fq.gz" and "Pop1_Sample1.R.fq.gz".

**Setting dDocent Options:**  
**Identifying the number of individuals in analysis**

```
output:
dDocent 2.2.7 
Contact jpuritz@gmail.com with any problems 
Checking for required software
All required software is installed!
dDocent run started Sat Dec 17 23:28:22 EST 2016 
20 individuals are detected. Is this correct? Enter yes or no and press [ENTER]
$ yes # only if this the number of individuals is correct. otherwise it is likely you're in the wrong directory
```
**Setting a hard limit for the number of processors**

```
output:  
dDocent detects 80 processors available on this system.  
Please enter the maximum number of processors to use for this analysis.
$ 80 
```
**Setting a hard limit for amount of memory use**  
This maximum memory limit only applies to SNP calling.

```
output:
dDocent detects 503 gigabytes of maximum memory available on this system.
Please enter the maximum memory to use for this analysis in gigabytes
For example, to limit dDocent to ten gigabytes, enter 10
This option does not work with all distributions of Linux.  If runs are hanging at variant calling, enter 0
$ 0
```
**Quality Trimming Reads**  
If this is the first time using dDocent on this data set, reads must be trimmed. If running dDocent on data files multiple times, reads don't need to be trimmed again.

```
output:
Do you want to quality trim your reads?
Type yes or no and press [ENTER]?
$ yes 
```

**Performing an Assembly**  
This option refers to *de novo* assembly. If you already have your own reference file or dDocent has already been used to assemble, then answer no. This project pipeline uses the *P. generosa* reference genome found [here](https://robertslab.github.io/sams-notebook/2019/01/15/Annotation-Geoduck-Genome-with-MAKER-Submitted-to-Mox.html).

```
output:
Do you want to perform an assembly?
Type yes or no and press [ENTER].
$ no 
```
If yes, see the full user guide for dDocent for instructions to perform as assembly: [dDocent](http://www.ddocent.com/UserGuide/#running).

> *de novo* assembly refers to the process of creating a genome, transcriptome, or proteome without the aid of a reference.

**Read Mapping**  
Read mapping has to be performed at least once before SNP calling. If trimming and assembly remains the same when using dDocent repeatly, then read mapping is not necessary. A (match score), B (mismatch score), and 0 (gap penalty) parameters are used. 

See [Background_Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md) for visual explanations of read mapping. 

*Match Score*: score for a matching base.

*Mismatch Score*: score for a mismatching base. 

*Gap Penalty*: Mutations, caused by either insertions or deletions, are annotated as gaps in the sequence and refered to as indels. These gaps are represented as dashes in the alignment. Optimization of gap penalty allows for the most accurate alignment possible. This will avoid low scores in alignments. Too small of a gap penalty will prevent a high level of alignment, and too large of a gap penalty will prevent accurate alignments. 

```
output: 
Do you want to map reads?  Type yes or no and press [ENTER]
$ yes

output:
BWA will be used to map reads.  You may need to adjust -A -B and -O parameters for your taxa.
Would you like to enter a new parameters now? Type yes or no and press [ENTER]
$ yes

output:
Please enter new value for A (match score).  It should be an integer.  Default is 1.
$ 1

output:
Please enter new value for B (mismatch score).  It should be an integer.  Default is 4.
$ 4

output:
Please enter new value for O (gap penalty).  It should be an integer.  Default is 6.
$ 6

```
> Read mapping refers to aligning reads to a reference sequence. [dDocent](http://www.ddocent.com/UserGuide/#read-mapping) uses the program [BWA](http://bio-bwa.sourceforge.net/) to output a SAM (Sequence Alignment/Map) file, which is then converted to a BAM (compact) file using `SAMtools`. 
> The default settings are optimized for the human genome and are generally conservative. Finding optimal values for each data set is recommended. 

**Calling SNPs**

```
output:
Do you want to use FreeBayes to call SNPs?  Please type yes or no and press [ENTER]
$ yes

output:
Please enter your email address.  dDocent will email you when it is finished running.
Don't worry; dDocent has no financial need to sell your email address to spammers.
$ # enter preferred email address

output:
dDocent will require input during the assembly stage.  Please wait until prompt says it is safe to move program to the background.
Trimming reads and simultaneously assembling reference sequences
```
Allow dDocent to run for a short time until more input is needed.

> FreeBayes detects variants based on differences in haplotypes, including single nucleotide polymorphisms (SNPs; see [Background_Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md)), and indels (insertions and deletions). [FreeBayes](https://hbctraining.github.io/In-depth-NGS-Data-Analysis-Course/sessionVI/lessons/02_variant-calling.html). 

```
output:
dDocent has finished with an analysis in /home/estrand/Final_Project/1mis_Demultiplexed_Data 

dDocent started Mon May 6 09:49:42 EDT 2019 

dDocent finished Mon May 6 11:53:18 EDT 2019 
After filtering, kept 22995 out of a possible 281567 Sites 

Using FreeBayes to call SNPs
100% 79:0=0s 56                                                                                                                                                                             

Using VCFtools to parse TotalRawSNPS.vcf for SNPs that are called in at least 90% of individuals
``` 
> See [Background_Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md) for more information on Variant Call Format (VCF). 


**Preliminary Filtering SNPs**  
Ideal cut-off values keep as much diversity in the data as possible while cutting out sequences that are likely errors. 

REPLACE BELOW WITH GEODUCK DATA

```
output:
              Number of Unique Sequences with More than X Coverage (Counted within individuals)                 
                                                                                                                        
  70000 +-+---------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+---------+-+   
        +           +           +           +           +           +           +           +           +           +   
        |                                                                                                           |   
  60000 ******                                                                                                    +-+   
        |     ******                                                                                                |   
        |           ******                                                                                          |   
        |                 ******                                                                                    |   
  50000 +-+                     *****                                                                             +-+   
        |                            *                                                                              |   
        |                             ******                                                                        |   
  40000 +-+                                 *****                                                                 +-+   
        |                                        *                                                                  |   
        |                                         ******                                                            |   
  30000 +-+                                             *****                                                     +-+   
        |                                                    *                                                      |   
        |                                                     ******                                                |   
  20000 +-+                                                         ******                                        +-+   
        |                                                                 ******                                    |   
        |                                                                       ******                              |   
        |                                                                             ******                        |   
  10000 +-+                                                                                 ************          +-+   
        |                                                                                               *************   
        +           +           +           +           +           +           +           +           +           +   
      0 +-+---------+-----------+-----------+-----------+-----------+-----------+-----------+-----------+---------+-+   
        2           4           6           8           10          12          14          16          18          20  
                                                          Coverage                                                                                                                                                                              
Please choose data cutoff.  In essence, you are picking a minimum (within individual) coverage level for a read (allele) to be used in the reference assembly
$ 

output:
                               Number of Unique Sequences present in more than X Individuals                            
                                                                                                                        
  5500 +-+---------+-----------+-----------+-----------+------------+-----------+-----------+-----------+---------+-+   
       **          +           +           +           +            +           +           +           +           +   
  5000 +-*                                                                                                        +-+   
       |  **                                                                                                        |   
       |    *                                                                                                       |   
  4500 +-+                                                                                                        +-+   
       |     ***                                                                                                    |   
  4000 +-+      **                                                                                                +-+   
       |          *                                                                                                 |   
  3500 +-+         *****                                                                                          +-+   
       |                *                                                                                           |   
  3000 +-+               *****                                                                                    +-+   
       |                      *                                                                                     |   
       |                       ******                                                                               |   
  2500 +-+                           ******                                                                       +-+   
       |                                   *****                                                                    |   
  2000 +-+                                      *                                                                 +-+   
       |                                         *************                                                      |   
  1500 +-+                                                    ******                                              +-+   
       |                                                            ************                                    |   
       |                                                                        ************                        |   
  1000 +-+                                                                                  ************          +-+   
       +           +           +           +           +            +           +           +           *************   
   500 +-+---------+-----------+-----------+-----------+------------+-----------+-----------+-----------+---------+-+   
       2           4           6           8           10           12          14          16          18          20  
                                                   Number of Individuals                                                
                                                                                                                        
Please choose data cutoff.  Pick point right before the asymptote. A good starting cutoff might be 10% of the total number of individuals
$

output:
At this point, all configuration information has been entered and dDocent may take several hours to run.
It is recommended that you move this script to a background operation and disable terminal input and output.
All data and logfiles will still be recorded.
To do this:
Press control and Z simultaneously
Type 'bg' without the quotes and press enter
Type 'disown -h' again without the quotes and press enter
```
All settings in dDocent have been set, now just wait for the program to finish. This may take several hours. 

### 5. In-depth Filtering SNPs
dDocent only performs preliminary filtering steps. The following steps include more in-depth filtering options.  

**Creating a new directory for Filtered SNPs**  

```
$ mkdir Filtered_Data
$ cd Filtered_Data
$ ln -s ../TotalRawSNPs.vcf # create symbolic link in the output file from dDocent
```

**Filerting by: missing data, alleles with low frequency, and alleles with low quality scores**  
Max-missing is ____. Maf is _____. MinQ is _____. 

```
$ vcftools --vcf TotalRawSNPs.vcf --max-missing 0.5 --maf 0.001 --minQ 20 --recode --recode-INFO-all --out TRS

output:
VCFtools - 0.1.14
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
        --vcf TotalRawSNPs.vcf
        --recode-INFO-all
        --maf 0.001
        --minQ 20
        --max-missing 0.5
        --out TRS
        --recode

After filtering, kept X out of X Individuals
Outputting VCF file...
After filtering, kept X out of a possible X Sites
Run Time = X seconds
```
**Filter out loci with low depth**  
minDP is _____.  
pop\_missing\_filter.sh is _____. 

```
$ vcftools --vcf TRS.recode.vcf --minDP 5 --recode --recode-INFO-all --out TRSdp5

output:
VCFtools - 0.1.14
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
        --vcf TRS.recode.vcf
        --recode-INFO-all
        --minDP 5
        --out TRSdp5
        --recode

After filtering, kept X out of X Individuals
Outputting VCF file...
After filtering, kept X out of a possible X Sites
Run Time = 3.00 seconds

$ pop_missing_filter.sh TRSdp5.recode.vcf ../popmap 0.05 1 TRSdp5p05

output:
After filtering, kept 80 out of 80 Individuals
Outputting VCF file...
After filtering, kept 2816 out of a possible 2993 Sites
Run Time = 3.00 seconds
```

**Filtering by: poor allelic balance, off kilter mapping quality ratios among alleles, etc.**  

What is the etc? What is dDocent doing?  
vcfallelicprimitives is ______.

```
$ dDocent_filters TRSdp5p05.recode.vcf TRSdp5p05

output:
                                            Histogram of mean depth per site

  250 +-+-+---+---+---+---+---+---+--+---+---+---+---+---+---+---+---+---+---+---+---+---+--+---+---+---+---+---+-+-+
      +   +   +   +   +   +   +   +  +   +   +   +   +   +   +   +   +   +   +   +   +   +  +   +   +   +   +   +   +
      |                                      *            'meandepthpersite' using (bin($1,binwidth)):(1.0) ******* |
      |                                     ***                                                                     |
      |                                     ***                                                                     |
  200 +-+                                  ****                                                                   +-+
      |                                    ****                                                                     |
      |                                   *****                                                                     |
      |                                   *****                                                                     |
  150 +-+                                *******                                                                  +-+
      |                                  *******                                                                    |
      |                                  *******                                                                    |
      |                                  ********                                                                   |
      |                                  ********                                                                   |
  100 +-+                                ********                                                                 +-+
      |                                  ********                                                                   |
      |                                  ********                                                                   |
      |                                **********                                                                   |
   50 +-+                              **********                                                                 +-+
      |                               ************                                                                  |
      |                              **************                                                                 |
      |                              ***************                                                                |
      +   +   +   +   +   +   +   +  ****************+   +   +   +   +   +   +   +   +   +  +   +   +   +   +   +   +
    0 +-+-+---+---+---+---+---+---+********************--+---+---+---+---+---+---+---+---+--+---+---+---+---+---+-+-+
      10  15  20  25  30  35  40  45 50  55  60  65  70  75  80  85  90  95 100 105 110 11 120 125 130 135 140 145 150
                                                        Mean Depth

If distrubtion looks normal, a 1.645 sigma cutoff (~90% of the data) would be 5171.64712
The 95% cutoff would be 64
Would you like to use a different maximum mean depth cutoff than 64, yes or no
$ 

output:
Please enter new cutoff
$ 

output:
Number of sites filtered based on maximum mean depth
 0 of 2136

Number of sites filtered based on within locus depth mismatch
 0 of 1976

Total number of sites filtered
 840 of 2816

Remaining sites
 1976

Filtered VCF file is called Output_prefix.FIL.recode.vcf

Filter stats stored in TRSdp5p05.filterstats

$ vcfallelicprimitives -k -g TRSdp5p05.FIL.recode.vcf |sed 's:\.|\.:\.\/\.:g' > TRSdp5p05F.prim
```

**Removing indels**  
Indels are ____. 

```
$ vcftools --vcf TRSdp5p05F.prim --recode --recode-INFO-all --remove-indels --out SNP.TRSdp5p05F

output:
After filtering, kept 80 out of 80 Individuals
Outputting VCF file...
After filtering, kept 1731 out of a possible 2130 Sites
Run Time = 1.00 seconds
```

**Filter by Hardy-Weinberg Equilibrium**  
Hardy-Weinberg Equilibrium is ____.

```
$ filter_hwe_by_pop.pl -v SNP.TRSdp5p05F.recode.vcf -p ../popmap -c 0.5 -out SNP.TRSdp5p05FHWE

output:
Processing population: PopA (20 inds)
Processing population: PopB (20 inds)
Processing population: PopC (20 inds)
Processing population: PopD (20 inds)
Outputting results of HWE test for filtered loci to 'filtered.hwe'
Kept 1731 of a possible 1731 loci (filtered 0 loci)
```

**Filter out low minor allele fre	quency (maf) loci** 
 
```
$ vcftools --vcf SNP.TRSdp5p05FHWE.recode.vcf --maf 0.05 --recode --recode-INFO-all --out SNP.TRSdp5p05FHWEmaf05

output:
VCFtools - 0.1.14
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
        --vcf SNP.TRSdp5p05FHWE.recode.vcf
        --recode-INFO-all
        --maf 0.05
        --out SNP.TRSdp5p05FHWEmaf05
        --recode

After filtering, kept 80 out of 80 Individuals
Outputting VCF file...
After filtering, kept 941 out of a possible 1731 Sites
```

**Rename final SNP vcf file for ease of use**

```
$ SNP.TRSdp5p05FHWE.recode.vcf final_SNPs.vcf
```


### Subtracting RADseq reads from EpiRADseq reads 

**Separating Epi and ddRAD into two .vcf files**

```
```

**Identifying loci with 0 reads in ddRAD set** 

```
```

**Removing those loci from EpiRAD set**

```
```