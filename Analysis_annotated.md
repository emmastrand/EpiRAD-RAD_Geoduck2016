# Visual Analysis   

Author: Emma Strand  
Last edited: 20190509

I struggled to get through all of the analysis for this project but below is the code and information for what I would have done. The goal was to do similar analysis to Dimond et al. 2017. 

Data uploaded and analyzed on KITT and R Studio. User logged in before steps are completed.  

See the [Bioinformatic Pipeline](https://github.com/emmastrand/EpiRAD-RAD_Geoduck2016/blob/master/Bioinformatic_Pipeline_annotated.md) page to view the data filtering steps preformed prior to the final visual analysis. 

## ddRAD and EpiRAD Data Yield

I was going to try to follow Jay Dimond's example:

```
#################################################################
# Now use edgeR package to standardize count data by library size

library("edgeR")
#read in the file to edgeR
counts <- DGEList(counts=data5)
counts$samples
#TMM normalization (corrects for library size)
counts2 <- calcNormFactors(counts)
counts2$samples
#extract normalized counts
counts2_cpm <- cpm(counts2, normalized.lib.sizes=TRUE, log=TRUE)

##Plots to show ddRAD vs EpiRAD library (before normalization)
par(mfrow = c(5, 5))
par(mar = c(2, ,2 ,2 ,2), oma = c(4, 4, 0.5, 0.5))

for (i in seq(1,49, by = 2)){
  plot(data5[,i], data5[,i+1], main = colnames(data5[i]))
}


#plot normalized counts
par(mfrow = c(5, 5))
par(mar = c(2, ,2 ,2 ,2), oma = c(4, 4, 0.5, 0.5)) 

for (i in seq(1,49, by = 2)){
  plot(counts2_cpm[,i], counts2_cpm[,i+1], main = colnames(counts2_cpm[i]))
}

```
The end goal would be one graph that had EpiRAD read counts on the y-axis and ddRAD read counts on the x-axis. Methylated reads would be shown at the bottom with a 0 value for EpiRAD but a read count for ddRAD. 

## DNA Methylation Heat Map 
I found a heat map script I was going to try from [https://sebastianraschka.com/Articles/heatmaps_in_r.html](https://sebastianraschka.com/Articles/heatmaps_in_r.html).

```
#########################################################
### A) Installing and loading required packages
#########################################################

if (!require("gplots")) {
   install.packages("gplots", dependencies = TRUE)
   library(gplots)
   }
if (!require("RColorBrewer")) {
   install.packages("RColorBrewer", dependencies = TRUE)
   library(RColorBrewer)
   }


#########################################################
### B) Reading in data and transform it into matrix format
#########################################################

data <- read.csv("../datasets/heatmaps_in_r.csv", comment.char="#")
rnames <- data[,1]                            # assign labels in column 1 to "rnames"
mat_data <- data.matrix(data[,2:ncol(data)])  # transform column 2-5 into a matrix
rownames(mat_data) <- rnames                  # assign row names


#########################################################
### C) Customizing and plotting the heat map
#########################################################

# creates a own color palette from red to green
my_palette <- colorRampPalette(c("red", "yellow", "green"))(n = 299)

# (optional) defines the color breaks manually for a "skewed" color transition
col_breaks = c(seq(-1,0,length=100),  # for red
  seq(0.01,0.8,length=100),           # for yellow
  seq(0.81,1,length=100))             # for green

# creates a 5 x 5 inch image
png("../images/heatmaps_in_r.png",    # create PNG for the heat map        
  width = 5*300,        # 5 x 300 pixels
  height = 5*300,
  res = 300,            # 300 pixels per inch
  pointsize = 8)        # smaller font size

heatmap.2(mat_data,
  cellnote = mat_data,  # same data set for cell labels
  main = "Correlation", # heat map title
  notecol="black",      # change font color of cell labels to black
  density.info="none",  # turns off density plot inside color legend
  trace="none",         # turns off trace lines inside the heat map
  margins =c(12,9),     # widens margins around plot
  col=my_palette,       # use on color palette defined earlier
  breaks=col_breaks,    # enable color transition at specified limits
  dendrogram="row",     # only draw a row dendrogram
  Colv="NA")            # turn off column clustering

dev.off()               # close the PNG device
```
 
## Discriminant Analysis of Principal Components (DAPC)

#### In R Studio
See the [EpiRAD_Geoduck.Rmd]( ) for R script.

**Load the necessary libraries**

[Adegenet](https://cran.r-project.org/web/packages/adegenet/adegenet.pdf) is an R software package that is specifically designed for handling genetic and genomic data. Applications include ploidy and heirarchical population structure ('genind'), allele counts by population ('genpop'), and genome-wide SNP data ('genlight'). As well as multivariate methods like DAPC and sPCA. 


```
library(adegenet)
library(vcfR)

my_vcf <- read.vcfR("final_SNPs.vcf")
my_genind <- vcfR2genind(my_vcf)

grp <- find.clusters(my_genind, max.n.clust=40)
table(pop(my_genind), grp$grp)

table.value(table(pop(my_genind), grp$grp), col.lab=paste("inf", 1:2), row.lab=paste("ori", 1:4))

dapc1 <- dapc(my_genind, grp$grp)
scatter(dapc1,col=col,bg="white", solid=1)

contrib <- loadingplot(dapc1$var.contr, axis=1, thres=.01, lab.jitter=1)
contrib

setPop(my_genind) <- ~Library

dapc1 <- dapc(my_genind, pop(my_genind))
contrib <- loadingplot(dapc1$var.contr, axis=1, thres=.05, lab.jitter=1)
```

## Principal Components Analysis (PCA)

I was going to try to follow Jay Dimond's example below:

```
########################################################
#PCA
pca <- prcomp(resid_t)
summary(pca)
eig <- (pca$sdev)^2

#plot
biplot(pca)

# Correlation between variables and principal components
var_cor_func <- function(var.loadings, comp.sdev){
  var.loadings*comp.sdev
}
# Variable correlation/coordinates
loadings <- pca$rotation
sdev <- pca$sdev
var.coord <- var.cor <- t(apply(loadings, 1, var_cor_func, sdev))
head(var.coord[, 1:4])

# Plot the correlation circle
a <- seq(0, 2*pi, length = 100)
plot( cos(a), sin(a), type = 'l', col="gray",
      xlab = "PC1",  ylab = "PC2")
abline(h = 0, v = 0, lty = 2)
# Add active variables
arrows(0, 0, var.coord[, 1], var.coord[, 2], 
       length = 0.1, angle = 15, code = 2)
# Add labels
text(var.coord, labels=rownames(var.coord), cex = 1, adj=1)
```

