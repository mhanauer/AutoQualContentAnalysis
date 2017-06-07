---
title: "AutoContentAnalysis"
output: html_document
---

Ok so instead of using numbers we can use categories and then assign certain texts to categories and then have the computer do the rest.

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here is the example provided by the manual.
```{r}
library(devtools)
install_github("iqss-research/VA-package")
install_github("iqss-research/ReadMeV1")
library(ReadMe)

demo(clinton)
setwd("~/Desktop")
setwd(system.file("QualAuto", package="ReadMe"))
undergrad.results = undergrad(sep = ',')

undergrad.preprocess <- preprocess(undergrad.results)
undergrad.preprocess$trainingset$FILENAME
readme.results <- readme(undergrad.preprocess)
setwd(oldwd)
readme.results$true.CSMF


```
Try to load data into the package.  Need to create a datafile that can be read into the package. 

It seems like we need a training and test data set.  It looks like we need numbers for the "Truth" of the training set 
Grabbing the original data from the qual file.  And grabbing the barriers  variables for each district and putting them together. 

```{r}
setwd("~/Desktop/QualData")
rbbcsc = read.csv("RBBCSCStaffSurvey.csv", header = TRUE); head(rbbcsc)
rbbcsc = as.data.frame(rbbcsc); head(rbbcsc)
mccsc = read.csv("MCCSCStaffSurvey.csv", header = TRUE); head(mccsc)

rbbcsc2 = rbbcsc[c("Q3")]
rbbcsc2 = as.data.frame(rbbcsc2)

mccsc2 = mccsc[c("Q3")]
mccsc2 = as.data.frame(mccsc2)

both = rbind(mccsc2, rbbcsc2)
write.csv(both, "both.csv")
both = read.csv("both.csv", header= TRUE, na.strings = c("", "NA"))
both = na.omit(both)
write.csv(both, "both.csv")
```
Here we are trying to clean the data set and create one that can be anlayzed by the read.me package.  Need three columns, filename, truth and trainingset.  Also transform the file into a sep-separated file.

First we need to take each of the word files and turn them into text files

```{r}
library(devtools)
source_url("https://gist.github.com/benmarwick/9266072/raw/csv2txts.R")
csv2txt("~/Desktop/QualAuto", labels = 1)
```
Ok now we have the data in text files, we need to create a date file with three columns, filename, truth, and trainingset.  So how do I put the files together?  First what does undergrad do?  Ok first need to create a filename with the path and name for each of the variables  


```{r}
both = both[c("Q3")]
names(both) = c("~/Desktop/QualAuto")
head(both)

set.seed(1)
Season <- replicate(405, rnorm(1, 0, 1))  # the returned object is a matrix
Season = as.data.frame(t(Season))
dim(Season)
names(Season) <- paste0("~/Desktop/QualAuto/", 1:ncol(Season), ".txt")
library(reshape2)
both1 = melt(Season, id.vars = 1)
both1 = both1$variable
head(both1)


undergrad.results = undergrad(sep = ',')

undergrad.preprocess <- preprocess(undergrad.results)
undergrad.preprocess$trainingset$FILENAME
readme.results <- readme(undergrad.preprocess)
```

