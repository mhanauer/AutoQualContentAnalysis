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

oldwd <- getwd()
setwd(system.file("demofiles/clintonposts", package="ReadMe"))
undergrad.results <- undergrad(sep = ',')
undergrad.preprocess <- preprocess(undergrad.results)
readme.results <- readme(undergrad.preprocess)
setwd(oldwd)


```
Here we are trying to clean the data set and create one that can be anlayzed by the read.me package.  Need three columns, filename, truth and trainingset.  Also transform the file into a sep-separated file.

First we need to take each of the word files and turn them into text files

```{r}
library(devtools)
source_url("https://gist.github.com/benmarwick/9266072/raw/csv2txts.R")
csv2txt("~/Desktop/QualAuto", labels = 1)
```


Try to load data into the package.  Need to create a datafile that can be read into the package. 

It seems like we need a training and test data set.  It looks like we need numbers for the "Truth" of the training set 
Grabbing the original data from the qual file.  And grabbing the barriers  variables for each district and putting them together. 

```{r}
setwd("~/Desktop/QualData")
rbbcsc = read.csv("RBBCSCStaffSurvey.csv", header = TRUE); head(rbbcsc)
rbbcsc = as.data.frame(rbbcsc); head(rbbcsc)
mccsc = read.csv("MCCSCStaffSurvey.csv", header = TRUE); head(mccsc)

mccsc2 = mccsc[c("Q3")]
mccsc2 = mccsc2[-c(1:2), ]
mccsc2 = as.matrix(mccsc2)


rbbcsc1 = rbbcsc[c("Q3")]
rbbcsc2 = rbbcsc1[-c(1:2), ]
rbbcsc2 = as.matrix(rbbcsc2)


both = rbind(mccsc2, rbbcsc2)
dim(both)

both = rbind(mccsc2, rbbcsc2)
write.csv(both, "both.csv")
both = read.csv("both.csv", header= TRUE, na.strings = c("", "NA"))
both = na.omit(both)
id = 1:nrow(both)
both = cbind(id, both)
both = both[c(1,3)]
write.csv(both, "both.csv", row.names = FALSE)
```

Ok now we have the data in text files, we need to create a date file with three columns, filename, truth, and trainingset.  So how do I put the files together?  First what does undergrad do?  Ok first need to create a filename with the path and name for each of the variables  


```{r}
both = both[c("Q3")]
names(both) = c("~/Desktop/QualAuto")
head(both)
# So here we are creating a dataset the same number values as the both data set
set.seed(1)
both1 <- replicate(405, rnorm(1, 0, 1))  # the returned object is a matrix
# Need to transpose this for some reason not sure why maybe get the columns in the right order?
both1 = as.data.frame(t(both1))
# Here we are pasting the pieces together
dim(both1)
names(both1) <- paste0( "/Users/matthewhanauer/Desktop/QualAuto/",1:404, ".txt")
library(reshape2)
# Now we taking what were the column headers and placing into a variable, but we lose the first value so need to add the back
both1 = melt(both1, id.vars = 1)
both1 = both1$variable
# We are getting only the variable that we want and renaming it.
write.csv(both1, "both1.csv")
both1 = read.csv("both1.csv", header = TRUE)
both1 = as.data.frame(both1)
both1 = both1[c("x")]
colnames(both1) = c("filename")
filename = both1$filename
filename = as.data.frame(filename)
dim(filename)
```
Now we need to create the file the can be read by the readme package
```{r}
# Need three sets of truth.  One for the possible values which are 1 and 2 in this case and then rest are NA's so we can cbind them later without an problems.
truth1 =  data.frame(a = rep(c(1,2),25))
truth1 = as.data.frame(truth1)
truth2 = data.frame(a = rep("", 404-50))
truth2 = as.data.frame(truth2)
truth = rbind(truth1, truth2)
names(truth) = c("truth")
truth$truth
trainingset1 = data.frame(a = rep(1,50))
trainingset1 = as.data.frame(trainingset1)
trainingset2 = data.frame(a = rep(0, 404-50))
trainingset2 = as.data.frame(trainingset2)
trainingset = rbind(trainingset1, trainingset2)
names(trainingset) = c("trainingset")
control = cbind(filename, truth, trainingset)
control
write.table(control, "control.txt", row.names = FALSE, sep = ",")
library(ReadMe)
head(control)
```
Now we need to get the undgrad function to work.

Two ways to possibly fix this.  First is figure out how to create a comma separated txt file 

Second is to figure out how they are setting the directory.
```{r}

setwd("~/Desktop/QualAuto")
undergrad.results = undergrad(sep = ',')
undergrad.preprocess <- preprocess(undergrad.results)
undergrad.preprocess$trainingset$FILENAME
readme.results <- readme(undergrad.preprocess)
```

