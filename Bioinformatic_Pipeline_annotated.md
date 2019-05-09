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
$ for i in *.F.fq.gz *.R.fq.gz 
> do print echo $(zcat $i|wc -l)/4|bc > Raw_Reads.txt
> done

echo $(zcat *.F.fq.gz *.R.fq.gz|wc -l)/4|bc

output:

```
> Each read in a fastq file has 4 lines: 1. identifier 2. sequence 3. description that starts with "+" 4. quality for each base in the second line. Raw reads can be counted by calculating the number of lines in the file and dividing by 4. `echo	` is used to start an argument and writes the argument as a standard output.`zcat` is used for zipped files. `bc` is used for calculations.

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

dDocent uses fastp to ____. 

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
At this point, all configuration information has been entered and dDocent may take several hours to run.
It is recommended that you move this script to a background operation and disable terminal input and output.
All data and logfiles will still be recorded.
To do this:
Press control and Z simultaneously
Type 'bg' without the quotes and press enter
Type 'disown -h' again without the quotes and press enter
```

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

dd_Geo29.F.fq.gz:102287
dd_Geo29.R1.fq.gz:101608
dd_Geo29.R2.fq.gz:101608
dd_Geo29.R.fq.gz:102287
dd_Geo30.F.fq.gz:185855
dd_Geo30.R1.fq.gz:183635
dd_Geo30.R2.fq.gz:183635
dd_Geo30.R.fq.gz:185855
dd_Geo32.F.fq.gz:201889
dd_Geo32.R1.fq.gz:198545
dd_Geo32.R2.fq.gz:198545
dd_Geo32.R.fq.gz:201889
dd_Geo33.F.fq.gz:285472
dd_Geo33.R1.fq.gz:278051
dd_Geo33.R2.fq.gz:278051
dd_Geo33.R.fq.gz:285472
dd_Geo34.F.fq.gz:529144
dd_Geo34.R1.fq.gz:526430
dd_Geo34.R2.fq.gz:526430
dd_Geo34.R.fq.gz:529144
dd_Geo35.F.fq.gz:40100
dd_Geo35.R1.fq.gz:39757
dd_Geo35.R2.fq.gz:39757
dd_Geo35.R.fq.gz:40100
dd_Geo3.F.fq.gz:1254140
dd_Geo3.R1.fq.gz:1244096
dd_Geo3.R2.fq.gz:1244096
dd_Geo3.R.fq.gz:1254140
dd_Geo4.F.fq.gz:1678591
dd_Geo4.R1.fq.gz:1664806
dd_Geo4.R2.fq.gz:1664806
dd_Geo4.R.fq.gz:1678591
dd_Geo60.F.fq.gz:287135
dd_Geo60.R1.fq.gz:281385
dd_Geo60.R2.fq.gz:281385
dd_Geo60.R.fq.gz:287135
dd_Geo66.F.fq.gz:48197
dd_Geo66.R1.fq.gz:47867
dd_Geo66.R2.fq.gz:47867
dd_Geo66.R.fq.gz:48197
Epi_Geo29.F.fq.gz:136608
Epi_Geo29.R1.fq.gz:135534
Epi_Geo29.R2.fq.gz:135534
Epi_Geo29.R.fq.gz:136608
Epi_Geo30.F.fq.gz:173525
Epi_Geo30.R1.fq.gz:171594
Epi_Geo30.R2.fq.gz:171594
Epi_Geo30.R.fq.gz:173525
Epi_Geo32.F.fq.gz:216549
Epi_Geo32.R1.fq.gz:212909
Epi_Geo32.R2.fq.gz:212909
Epi_Geo32.R.fq.gz:216549
Epi_Geo33.F.fq.gz:340727
Epi_Geo33.R1.fq.gz:337985
Epi_Geo33.R2.fq.gz:337985
Epi_Geo33.R.fq.gz:340727
Epi_Geo34.F.fq.gz:472694
Epi_Geo34.R1.fq.gz:470297
Epi_Geo34.R2.fq.gz:470297
Epi_Geo34.R.fq.gz:472694
Epi_Geo35.F.fq.gz:41724
Epi_Geo35.R1.fq.gz:41351
Epi_Geo35.R2.fq.gz:41351
Epi_Geo35.R.fq.gz:41724
Epi_Geo3.F.fq.gz:1618046
Epi_Geo3.R1.fq.gz:1604135
Epi_Geo3.R2.fq.gz:1604135
Epi_Geo3.R.fq.gz:1618046
Epi_Geo4.F.fq.gz:1602472
Epi_Geo4.R1.fq.gz:1589114
Epi_Geo4.R2.fq.gz:1589114
Epi_Geo4.R.fq.gz:1602472
Epi_Geo60.F.fq.gz:326689
Epi_Geo60.R1.fq.gz:321284
Epi_Geo60.R2.fq.gz:321284
Epi_Geo60.R.fq.gz:326689
Epi_Geo66.F.fq.gz:50947
Epi_Geo66.R1.fq.gz:50506
Epi_Geo66.R2.fq.gz:50506
Epi_Geo66.R.fq.gz:50947
``` 
Name.F.fq.gz = # of SNPs in original forward sequence   
Name.R1.fq.gz	 = # of SNPs in filtered forward seqeuence  
Name.R.fq.gz	 = # of SNPs in original reverse sequence  
Name.R2.fq.gz = # of SNPs in filtered reverse sequence
 

**Using samtools to check file stats**

In order to decide what mapping parameters to use, we can compare percentage mapped (line 5), percentage properly paired (line 9), percentage of singletons (line 11), and number mapped to a different chromosome (line 13). A singleton refers to reads that had one of the paired end reads mapped to the genome and the other did not. "With itself and mate mapped" indicates that both reads mapped to the genome.

```
# running a for loop to calculate stats on each file.
$ for i in *.bam
> do samtools flagstat $i > $i.txt
> done
$ mkdir 1_4_6_flagstats # in case I decided to go back and change the mapping parameters
$ cd 1_4_6_flagstats
$ cat *.txt >> 146flagstats.txt

# View the stats altogether 
$ cat 146flagstats.txt 

# Or one file at a time
$ cat dd_Geo29-RG.bam.txt

output:
134154 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
134154 + 0 mapped (100.00% : N/A)
134154 + 0 paired in sequencing
67511 + 0 read1
66643 + 0 read2
122065 + 0 properly paired (90.99% : N/A)
133935 + 0 with itself and mate mapped
219 + 0 singletons (0.16% : N/A)
7173 + 0 with mate mapped to a different chr
6596 + 0 with mate mapped to a different chr (mapQ>=5)

$ cat Epi_Geo29-RG.bam.txt

output:
174704 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
174704 + 0 mapped (100.00% : N/A)
174704 + 0 paired in sequencing
88016 + 0 read1
86688 + 0 read2
155537 + 0 properly paired (89.03% : N/A)
174327 + 0 with itself and mate mapped
377 + 0 singletons (0.22% : N/A)
13080 + 0 with mate mapped to a different chr
12299 + 0 with mate mapped to a different chr (mapQ>=5)

$ cat dd_Geo30-RG.bam.txt

output:
248279 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
248279 + 0 mapped (100.00% : N/A)
248279 + 0 paired in sequencing
125358 + 0 read1
122921 + 0 read2
226069 + 0 properly paired (91.05% : N/A)
248082 + 0 with itself and mate mapped
197 + 0 singletons (0.08% : N/A)
16163 + 0 with mate mapped to a different chr
15100 + 0 with mate mapped to a different chr (mapQ>=5)

$ cat Epi_Geo30-RG.bam.txt

output:
222078 + 0 in total (QC-passed reads + QC-failed reads)
0 + 0 secondary
0 + 0 supplementary
0 + 0 duplicates
222078 + 0 mapped (100.00% : N/A)
222078 + 0 paired in sequencing
112118 + 0 read1
109960 + 0 read2
191872 + 0 properly paired (86.40% : N/A)
221881 + 0 with itself and mate mapped
197 + 0 singletons (0.09% : N/A)
23548 + 0 with mate mapped to a different chr
22392 + 0 with mate mapped to a different chr (mapQ>=5)

# and so on for each sample
```

### 5. In-depth Filtering SNPs
dDocent only performs preliminary filtering steps. The following steps include more in-depth filtering options with [vcf tools](http://vcftools.sourceforge.net) following dDocent's [filtering tutorial](http://www.ddocent.com/filtering/). See [Background_Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md) for more information on Variant Call Format (VCF). [Flag guide to VCFtools](http://vcftools.sourceforge.net/man_latest.html). 

**Creating a new directory for Filtered SNPs**  

```
$ cd ..
$ mkdir Filtered_Data
$ cd Filtered_Data
$ ln -s ../1mis_Demultiplexed_Data/TotalRawSNPs.vcf # create symbolic link in the output file from dDocent
```

**Filerting by: missing data, alleles with minor counts, and alleles with low quality scores**  
Max-missing is filtering for missing data. 0.5 filters out genotypes that were called below 50% across individuals. In other words, this only keeps variants that were successfully genotyped in 50% of individuals. This value will be between 0 and 1.  

Mac is filtering for minor allele counts. Allele count is the number of times that allele appears over all individuals at that site. 3 requires that a allele appear in 1 homozygote and 1 heterozygote OR 3 heterozygotes.    

MinQ is filtering for alleles with low quality scores. 30 is set as the minimum quality score for each allele. 

```
$ vcftools --vcf TotalRawSNPs.vcf --max-missing 0.5 --mac 3 --minQ 30 --recode --recode-INFO-all --out SNPs

output:
VCFtools - 0.1.16
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf TotalRawSNPs.vcf
	--recode-INFO-all
	--mac 3
	--minQ 30
	--max-missing 0.5
	--out SNPs
	--recode

#several warning messages 

After filtering, kept 20 out of 20 Individuals
Outputting VCF file...
After filtering, kept 62183 out of a possible 281567 Sites
Run Time = 28.00 seconds
```
`recode` creates a new VCF file with the filters.  
`recode-INFO-all` keeps all the flags from the last file in the new one.  
`--out` indicates the name of the output

**Filtering by: minimum depth for genotype call and minimum mean depth**

Setting a mininmum depth of 3 will only keep genotypes that have 3 amount of reads and above. 

```
$ vcftools --vcf SNPs.recode.vcf --minDP 3 --recode --recode-INFO-all --out SNPs_dp3

output:
VCFtools - 0.1.16
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf SNPs.recode.vcf
	--recode-INFO-all
	--minDP 3
	--out SNPs_dp3
	--recode

# several warning messages 

After filtering, kept 20 out of 20 Individuals
Outputting VCF file...
After filtering, kept 62183 out of a possible 62183 Sites
Run Time = 13.00 seconds

```
> "Yes, we are keeping genotypes with as few as 3 reads. We talked about this in the lecture portion of this course, but the short answer is that sophisticated multisample variant callers like FreeBayes and GATK can confidently call genotypes with few reads because variants are assessed across all samples simultaneously. So, the genotype is based on three reads AND prior information from all reads from all individuals. Relax. We will do plenty of other filtering steps!" - dDocent  
> See dDocent's tutorials for more information if concerned.

**Evaluate potential errors**

Download the following script from dDocent.

```
$ curl -L -O https://github.com/jpuritz/dDocent/raw/master/scripts/ErrorCount.sh
$ chmod +x ErrorCount.sh 
$ ErrorCount.sh SNPs_dp3.recode.vcf

output:
This script counts the number of potential genotyping errors due to low read depth
It report a low range, based on a 50% binomial probability of observing the second allele in a heterozygote and a high range based on a 25% probability.
Potential genotyping errors from genotypes from only 1 read range from 0.0 to 0.0
Potential genotyping errors from genotypes from only 2 reads range from 0.0 to 0.0
Potential genotyping errors from genotypes from only 3 reads range from 4817.25 to 16185.96
Potential genotyping errors from genotypes from only 4 reads range from 1538.3125 to 7777.708
Potential genotyping errors from genotypes from only 5 reads range from 515.34375 to 3908
20 number of individuals and 62183 equals 1243660 total genotypes
Total genotypes not counting missing data 958009
Total potential error rate is between 0.0071720685818191686 and 0.029093325845581823
SCORCHED EARTH SCENARIO
WHAT IF ALL LOW DEPTH HOMOZYGOTE GENOTYPES ARE ERRORS?????
The total SCORCHED EARTH error rate is 0.08313283069365737.

```
`curl` function transfers data from or to a server  
`-L` tells curl to follow links  
`-O` tells curl to write a file name  
`chmod +x` changes the mode of the file and makes it executable

Our error rate because of genotypes under 5 reads is under 5%. 

**Filtering by: missing data from individuals**

There are 2 options for this step: the following code OR a script that runs this code that can be downloaded from dDocent. See the end of this section for the dDocent script instructions. 

This step filters out individuals that did not sequence well. The file generated reports the missing data on a per-individual basis. The output is an "imiss" file.

```
# creates imiss file
$ vcftools --vcf SNPs_dp3.recode.vcf --missing-indv

output:
VCFtools - 0.1.16
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf SNPs_dp3.recode.vcf
	--missing-indv

# several warning messages 

After filtering, kept 20 out of 20 Individuals
Outputting Individual Missingness
After filtering, kept 62183 out of a possible 62183 Sites
Run Time = 2.00 seconds

# views imiss file 
$ cat out.imiss

output:
INDV	N_DATA	N_GENOTYPES_FILTERED	N_MISS	F_MISS
dd_Geo29	62183	0	36624	0.588971
dd_Geo30	62183	0	25616	0.411945
dd_Geo32	62183	0	25561	0.411061
dd_Geo33	62183	0	22146	0.356142
dd_Geo34	62183	0	12933	0.207983
dd_Geo35	62183	0	54980	0.884164
dd_Geo3	62183	0	1908	0.0306836
dd_Geo4	62183	0	862	0.0138623
dd_Geo60	62183	0	22462	0.361224
dd_Geo66	62183	0	54572	0.877603
Epi_Geo29	62183	0	32396	0.520978
Epi_Geo30	62183	0	29856	0.480131
Epi_Geo32	62183	0	27228	0.437869
Epi_Geo33	62183	0	22237	0.357606
Epi_Geo34	62183	0	17547	0.282183
Epi_Geo35	62183	0	57281	0.921168
Epi_Geo3	62183	0	6255	0.10059
Epi_Geo4	62183	0	6001	0.0965055
Epi_Geo60	62183	0	23900	0.384349
Epi_Geo66	62183	0	54545	0.877169

```
`F_MISS` is the frequency of missing data for that individual (N_MISS/TOTAL SITES)  
`N_MISS` is the number of sites the individual does not have any data for

The highest percentage of missing data we have is 92.1% (Epi_Geo35).

To create a histogram of this data:

```
$ mawk '!/IN/' out.imiss | cut -f5 > totalmissing
$ gnuplot << \EOF 
> set terminal dumb size 120, 30
> set autoscale
> unset label
> set title "Histogram of % missing data per individual"
> set ylabel "Number of Occurrences"
> set xlabel "% of missing data"
> #set yr [0:100000]
> binwidth=0.01
> bin(x,width)=width*floor(x/width) + binwidth/2.0
> plot 'totalmissing' using (bin($1,binwidth)):(1.0) smooth freq with boxes 
> pause -1
> EOF

output:

                                        Histogram of % missing data per individual

    2 ++----------------+----------*****---***---------------+-----------***************----------+----------------++
      +                 +          *   *   * *               + 'totalmissing' using (bin($1,binwidth)):(1.0) ****** +
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
  1.8 ++                           *   *   * *                           *             *                           ++
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
  1.6 ++                           *   *   * *                           *             *                           ++
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
  1.4 ++                           *   *   * *                           *             *                           ++
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
  1.2 ++                           *   *   * *                           *             *                           ++
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      |                            *   *   * *                           *             *                            |
      +                 +          *   *   * *               +           *     +       *          +                 +
    1 ++----------------+----------*****---***---------------+-----------***************----------+----------------++
      0                0.2                0.4               0.6               0.8                 1                1.2
                                                     % of missing data


```
*This plot wouldn't work unless in conda environment. But most of the steps previously were not done in the conda environment. Come back to this. Plot is also missing ylabel. Plot doesn't really look like the data. Come back to this.*

*All samples were kept for the project (0.95 cutoff), but come back to this for publication. Could take out the samples above 70% missing data. 50% is a more conservative value, but we have sample sizes here. Come back to this.* 

Filtering out the individuals with high percentage of missing data:

```
# To make a list of individuals with more than 95% missing data
$ mawk '$5 > 0.95' out.imiss | cut -f1 > lowDP.indv

# Remove these individuals from the data set 
# adding "md" to the name of the file 
$ vcftools --vcf SNPs_dp3.recode.vcf --remove lowDP.indv --recode --recode-INFO-all --out SNPs_dp3_md

output:
VCFtools - 0.1.17
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf SNPs_dp3.recode.vcf
	--remove lowDP.indv
	--recode-INFO-all
	--out SNPs_dp3_md
	--recode

After filtering, kept 20 out of 20 Individuals
Outputting VCF file...
After filtering, kept 62183 out of a possible 62183 Sites
Run Time = 6.00 seconds
```

**OR**  
Filtering by individuals with low coverage can also be done with a script from dDocent. You only need to complete either the above code or the script from dDocent. The output will be the same.

```
$ curl -L -O https://github.com/jpuritz/dDocent/raw/master/scripts/filter_missing_ind.sh
$ chmod +x filter_missing_ind.sh
$ ./filter_missing_ind.sh (file to be filtered name) (new name prefix)

output: will be a histogram like the one above. The default cut off is 85% and you will have the option to change this if you'd like.  
Enter yes or no. 

Output file (new name prefix).vcf will be filtered by individuals with high percentage of missing data (individuals with low coverage).
```

**Filter by: mean depth of genotypes and variants with a high percentage of coverage across individuals**

Max-missing is filtering for missing data. 0.95 filters out SNPs that weren't called in 95% or higher of the individuals. The genotype call rate is 95% across all individuals.  

Maf is filtering by minor allele frequency. Allele frequency is the number of times an allele appears over all individuals at that site, divided by the total number of non-missing alleles at that site. 0.05 filters out alleles that do no occur in more than 5% of the individuals.  

min-meanDP filters for sites with mean depth values over all individuals that are greater than or equal to the min-meanDP value. 20 will filter out sites that have a depth values of less than 20.   

```
$ vcftools --vcf SNPs_dp3_md.recode.vcf --max-missing 0.95 --maf 0.05 --recode --recode-INFO-all --out SNPs_dp3_md_g95 --min-meanDP 20

output:
VCFtools - 0.1.17
(C) Adam Auton and Anthony Marcketta 2009

Parameters as interpreted:
	--vcf SNPs_dp3_md.recode.vcf
	--recode-INFO-all
	--maf 0.05
	--min-meanDP 20
	--max-missing 0.95
	--out SNPs_dp3_md_g95
	--recode

After filtering, kept 20 out of 20 Individuals
Outputting VCF file...
After filtering, kept 2475 out of a possible 62183 Sites
Run Time = 1.00 seconds
```

If you have more than 2 localities, it is best to filter by population call rate. See [dDocent](http://www.ddocent.com/filtering/) for more information on this. *Come back to this for the real analysis.*

To view the header of the vcf file:

```
$ mawk '/#/' SNPs_dp3_md_g95.recode.vcf

portion of the output:
##INFO=<ID=NS,Number=1,Type=Integer,Description="Number of samples with data">
##INFO=<ID=DP,Number=1,Type=Integer,Description="Total read depth at the locus">
##INFO=<ID=DPB,Number=1,Type=Float,Description="Total read depth per bp at the locus; bases in reads overlapping / bases in haplotype">
##INFO=<ID=AC,Number=A,Type=Integer,Description="Total number of alternate alleles in called genotypes">
##INFO=<ID=AN,Number=1,Type=Integer,Description="Total number of alleles in called genotypes">
##INFO=<ID=AF,Number=A,Type=Float,Description="Estimated allele frequency in the range (0,1]">
##INFO=<ID=RO,Number=1,Type=Integer,Description="Reference allele observation count, with partial observations recorded fractionally">
##INFO=<ID=AO,Number=A,Type=Integer,Description="Alternate allele observations, with partial observations recorded fractionally">
##INFO=<ID=PRO,Number=1,Type=Float,Description="Reference allele observation count, with partial observations recorded fractionally">
##INFO=<ID=PAO,Number=A,Type=Float,Description="Alternate allele observations, with partial observations recorded fractionally">
##INFO=<ID=QR,Number=1,Type=Integer,Description="Reference allele quality sum in phred">
##INFO=<ID=QA,Number=A,Type=Integer,Description="Alternate allele quality sum in phred">
##INFO=<ID=PQR,Number=1,Type=Float,Description="Reference allele quality sum in phred for partial observations">
##INFO=<ID=PQA,Number=A,Type=Float,Description="Alternate allele quality sum in phred for partial observations">
##INFO=<ID=SRF,Number=1,Type=Integer,Description="Number of reference observations on the forward strand">
##INFO=<ID=SRR,Number=1,Type=Integer,Description="Number of reference observations on the reverse strand">
##INFO=<ID=SAF,Number=A,Type=Integer,Description="Number of alternate observations on the forward strand">
##INFO=<ID=SAR,Number=A,Type=Integer,Description="Number of alternate observations on the reverse strand">

# the above information will help in deciding which filter steps are still needed
```

**Filter by: allele balance**

Allele balance is the ratio of the reference allele to all reads and only considers heterozygotes. This is a value between 0 and 1. RADseq targets specific locations of the genome so we expect the ratio to be close to 0.5. AB > 0.25 and AB < 0.75 filters out loci with a balance below 25 and above 75. AB < 0.01 will keep loci with fixed variants (homozygotes). 

```
# to view the options of vcffilter 
$ vcffilter

portion of the output:
options:
    -f, --info-filter     specifies a filter to apply to the info fields of records,
                          removes alleles which do not pass the filter
    -g, --genotype-filter specifies a filter to apply to the genotype fields of records
    -k, --keep-info       used in conjunction with '-g', keeps variant info, but removes genotype
    -s, --filter-sites    filter entire records, not just alleles
    -t, --tag-pass        tag vcf records as positively filtered with this tag, print all records
    -F, --tag-fail        tag vcf records as negatively filtered with this tag, print all records
    -A, --append-filter   append the existing filter tag, don't just replace it
    -a, --allele-tag      apply -t on a per-allele basis.  adds or sets the corresponding INFO field tag
    -v, --invert          inverts the filter, e.g. grep -v
    -o, --or              use logical OR instead of AND to combine filters
    -r, --region          specify a region on which to target the filtering, requires a BGZF
                          compressed file which has been indexed with tabix.  any number of
                          regions may be specified.
                          

# to filter by allele balance
$ vcffilter -s -f "AB > 0.25 & AB < 0.75 | AB < 0.01" SNPs_dp3_md_g95.recode.vcf > SNPs_dp3_md_g95_AB.fil1.vcf

# checking how many loci are in the pre AB-filter vcf 
$ mawk '!/#/' SNPs_dp3_md_g95.recode.vcf | wc -l

output: 2475

# checking how many loci are in the post AB-filter vcf
$ mawk '!/#/' SNPs_dp3_md_g95_AB.fil1.vcf | wc -l

output: 1031

```
`wc -l` counts the number of lines in a file

**Filtering by: reads on both strands**

A SNP should appear in only forward or only reverse reads.

```
$ vcffilter -f "SAF / SAR > 100 & SRF / SRR > 100 | SAR / SAF > 100 & SRR / SRF > 100" -s SNPs_dp3_md_g95_AB.fil1.vcf > SNPs_dp3_md_g95_AB_BS.fil2.vcf

# checking how many loci are in the post both strand (BS) filter vcf
$ mawk '!/#/' SNPs_dp3_md_g95_AB_BS.fil2.vcf | wc -l

output: 1024
```
This is filtering out loci that have less than 100 times more forward alternate reads than reverse alternate reads. And 100 times more reverse reference reads than forward reference reads. And the reciprocal of both. If the allele is identical to the reference then it is a reference allele, and if it is not then it is an alternate allele. These are likely to be errors in PCR, paralogs, or microbe contamination. 

SAR, SAF, and SRR are strand bias counts. *Can't find what they stand for. Come back to this.*

**Filter by: mapping quality between reference and alternate alleles**

Because RADseq loci and alleles are specific regions of the genome, the mapping quality between reference and alternate should be very similar.

```
$ vcffilter -f "MQM / MQMR > 0.9 & MQM / MQMR < 1.05" SNPs_dp3_md_g95_AB_BS.fil2.vcf > SNPs_dp3_md_g95_AB_BS_MQ.fil3.vcf

$ mawk '!/#/' SNPs_dp3_md_g95_AB_BS_MQ.fil3.vcf | wc -l

output: 806
```

**Filter by: properly paired status for reference and alternate alleles**

If the reads supporting the reference allele are properly paired, but the reads supporting the alternate are not, then it becomes a problem.

*Why? What is 0.05, 1.75, 0.25. Come back to this*

```
$ vcffilter -f "PAIRED > 0.05 & PAIREDR > 0.05 & PAIREDR / PAIRED < 1.75 & PAIREDR / PAIRED > 0.25 | PAIRED < 0.05 & PAIREDR < 0.05" -s SNPs_dp3_md_g95_AB_BS_MQ.fil3.vcf > SNPs_dp3_md_g95_AB_BS_MQ_PP.fil4.vcf

$ mawk '!/#/' SNPs_dp3_md_g95_AB_BS_MQ_PP.fil4.vcf | wc -l

output: 805
```

**Filter by: ratio of quality score to depth**

In whole genome sequences, high coverage can lead to inflated locus quality scores. See [dDocent](http://www.ddocent.com/filtering/), [Heng Li](http://arxiv.org/pdf/1404.0929.pdf), and [Brad Chapman](http://bcb.io/2014/05/12/wgs-trio-variant-evaluation/) for more information on this. 

dDocent recommends a less conservative model than Heng Li because RADseq loci are not radnomly distributed across contigs. There are two options below: 1.) code line by line 2.) script from dDocent 

Option 1:

```
## STEP 1
# to remove a locus with a quality score below 1/4 of the depth
$ vcffilter -f "QUAL / DP > 0.25" SNPs_dp3_md_g95_AB_BS_MQ_PP.fil4.vcf > SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf
$ mawk '!/#/' SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf | wc -l

output: 798

## STEP 2
# Create a list of depths at each locus 
$ cut -f8 SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf | grep -oe "DP=[0-9]*" | sed -s 's/DP=//g' > SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.DEPTH

# Create a list of quality scores 
$ mawk '!/#/' SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf | cut -f1,2,6 > SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf.loci.qual

# Calculate mean depth 
$ mawk '{ sum += $1; n++ } END { if (n > 0) print sum / n; }' SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.DEPTH
output: 1202.7

# Mean plus 3X the square of the mean
# the previous mawk output is used twice in the following 
$ python -c "print int(1202.7+3*(1202.7**0.5))"
output: 1306

# Paste depth and quality files together, find loci above the cutoff that don't have quality scores 2 times the depth
# x = the previous python output
$ paste SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf.loci.qual SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.DEPTH | mawk -v x=1306 '$4 > x' | mawk '$3 < 2 * $4' > SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.lowQDloci

# Remove those sites below the cutoff, recalculate depth across loci
$ vcftools --vcf SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf --site-depth --exclude-positions SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.lowQDloci --out SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5

output:
After filtering, kept 20 out of 20 Individuals
Outputting Depth for Each Site
After filtering, kept 790 out of a possible 798 Sites
Run Time = 0.00 seconds

# Creating a file of only depth scores
$ cut -f3 SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.ldepth > SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.site.depth

# Calculate average depth by diving above file by # of individuals
$ mawk '!/D/' SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.site.depth | mawk -v x=20 '{print $1/x}' > meandepthpersite

# Plotting this data as a histogram 
$ gnuplot << \EOF 
> set terminal dumb size 120, 30
> set autoscale
> set xrange [10:150] 
> unset label
> set title "Histogram of mean depth per site"
> set ylabel "Number of Occurrences"
> set xlabel "Mean Depth"
> binwidth=1
> bin(x,width)=width*floor(x/width) + binwidth/2.0
> set xtics 5
> plot 'meandepthpersite' using (bin($1,binwidth)):(1.0) smooth freq with boxes
> pause -1
> EOF 

output: 
                                            Histogram of mean depth per site

  40 ++--+---+---+---+---+---+---+---+---+---+---+---+---+---+--+---+---+---+---+---+---+---+---+---+---+---+---+--++
     +   +   +   +   +   +   +   +   +   +   +   +   +   + 'meandepthpersite' using (bin($1,binwidth)):(1.0)+****** +
     |                                  **                                                                          |
  35 ++                           *     **                                                                         ++
     |                            *     **                                                                          |
     |                          ***     **                                                                          |
  30 ++                         ***     **                                                                         ++
     |                        ***** **  **                                                                          |
     |                        ***** **  **                                                                          |
  25 ++                       ***** **  **                                                                         ++
     |                        ***** **  ****                                                                        |
  20 ++       **             *********  *****                                                                      ++
     |        **          *  ****************                                                                       |
     |        **      **  *******************                                                                       |
  15 ++       **      **  ********************                                                                     ++
     |        **      **  ********************                                                                      |
     |        **      **  ********************  **                                                                  |
  10 ++       **      ************************ *****         *          *                                          ++
     |        **      ******************************         *        ***                                           |
     |        **  *** *******************************      ***        ***                                           |
   5 ++       ** ** **********************************   * ***  **  * ***                                          ++
     |        ** ** ************************************** ***  ***** ***                                           |
     +   +   +***** ********************************** **************************************************************
   0 ++--+---+*******************************************************************************************************
     10  15  20  25  30  35  40  45  50  55  60  65  70  75  80 85  90  95 100 105 110 115 120 125 130 135 140 145 150
                                                       Mean Depth

```
Loci with high mean depth are likely paralogs or multicopy loci. 
Removing all loci with a high mean depth above 100:

```
$ vcftools --vcf  SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.vcf --recode-INFO-all --out SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.FIL --max-meanDP 100 --exclude-positions SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.fil5.lowQDloci --recode 

output:
After filtering, kept 20 out of 20 Individuals
Outputting VCF file...
After filtering, kept 740 out of a possible 798 Sites
Run Time = 0.00 seconds
```
OR  
Option 2:

```
$ curl -L -O https://github.com/jpuritz/dDocent/raw/master/scripts/dDocent_filters
$ chmod +x dDocent_filters
$ ./dDocent_filters

This script will automatically filter a FreeBayes generated VCF file using criteria related to site depth,
quality versus depth, strand representation, allelic balance at heterzygous individuals, and paired read representation.
The script assumes that loci and individuals with low call rates (or depth) have already been removed.

Contact Jon Puritz (jpuritz@gmail.com) for questions and see script comments for more details on particular filters

$ sh FB_filters.sh (VCF_file) (Output_prefix)
``` 

**Filter by: Hardy-Weinberg Equilibrium (HWE)**

SNPs with extreme violations of Hardy-Weinberg Equilibrium are likely due to technical artifacts. This filters out sites that have excessive heterozygotes that aren't reflective of real data. We can use a script from dDocent for this.

```
$ curl -L -O https://github.com/jpuritz/dDocent/raw/master/scripts/filter_hwe_by_pop.pl
$ chmod +x filter_hwe_by_pop.pl

# Convert variant calls to SNPs
# following code decomposes complex variant calls into phased SNP and INDEL genotypes, and keeps the INFO flags for loci and genotypes
$ vcfallelicprimitives SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.FIL.recode.vcf --keep-info --keep-geno > SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.prim.vcf

# viewing number of SNPs
$ mawk '!/#/' SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.prim.vcf | wc -l

output: 808

# Removing indels
$ vcftools --vcf SNPs_dp3_md_g95_AB_BS_MQ_PP_QS.prim.vcf --remove-indels --recode --recode-INFO-all --out SNP.dp3_md_g95_AB_BS_MQ_PP_QS

output:
After filtering, kept 20 out of 20 Individuals
Outputting VCF file...
After filtering, kept 744 out of a possible 808 Sites
Run Time = 0.00 seconds

# applying HWE filter
$ ./filter_hwe_by_pop.pl -v SNP.dp3_md_g95_AB_BS_MQ_PP_QS -p ../1mis_Demultiplexed_Data/popmap -o SNP.dp3_md_g95_AB_BS_MQ_PP_QS.HWE -h 0.01

output: 

```
*Number of SNPs increased?? Come back to this. Also come back to choosing a Hardy-Weinberg value. and cutoff value?*

`-v` is the input file  
`-o` is the output file  
`-p` is the popmap file that is an output file from dDocent  
`-h` is the minimum cutoff for Hardy-Weinberg p-value  
`-c` is a cuttoff value for the proportion of the population that a locus can be below HWE cutoff without being filtered. 0.5 would filter SNPs that are below the p-value threshold in 50% or more of the populations.

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