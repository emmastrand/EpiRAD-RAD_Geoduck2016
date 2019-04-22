## *P. generosa* RAD and EpiRADseq Bioinformatic Pipeline Anlaysis

Author: Emma Strand  
Last Edited: 20190422

Data upload and analyzed on KITT.
### Using Terminal
#### Set-Up (downloading software programs, creating folders, and uploading raw data)
Downloading Bioconda (skip this step if Bioconda has already been downloaded).

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

Create and activate an environment in conda  

```
$ conda create -n Final_Project
$ conda activate Final_Project
```

Creating and entering a directory for this project:  

```
$ mkdir Final_Project # creates a directory
$ cd Final_Project # navigates into this new directory
```

Copying raw data files to directory on KITT

```
$ scp -P XXXX 
# XXXX above indicates the password specific for our lab's KITT.
```

Installing and starting dDocent 

```
$ conda install dDocent # you only need to do this once. skip if already downloaded.
$ dDocent
```

> dDocent is a bioinformatics program created by [Dr. Jon Purtiz](https://github.com/jpuritz) that is specifically designed for different types of RAD sequencing. See the [Background Information](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Background_Information.md) file in this repository for more information.


