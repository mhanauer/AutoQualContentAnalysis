---
title: "AutoContentAnalysis"
output: html_document
---

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
oldwd <- getwd()
setwd(system.file("demofiles/clintonposts", package="ReadMe"))
undergrad.results = undergrad(sep = ',')

undergrad.preprocess <- preprocess(undergrad.results)
undergrad.preprocess$trainingset$FILENAME
readme.results <- readme(undergrad.preprocess)
setwd(oldwd)
readme.results


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
