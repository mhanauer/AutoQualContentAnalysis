---
title: "Automated Content Analysis in R"
output: html_document
---
```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```
Here is an example of using the Readme automated content analysis package in R.  Our goal is to take a set of survey responses to a question and categorize each of them into one of two responses titled 1 and 2.  The first step is to load, install, and library all of the necessary packages.
```{r}
library(devtools)
install_github("iqss-research/VA-package")
install_github("iqss-research/ReadMeV1")
library(ReadMe)
```
Here I am just cleaning the data.  The main point is that you want two columns.  One with a value that goes from 1 to the length of the rows in your data set and the actual text values in their own column. You will need to delete all of the columns from the data as these will confuse the next R package that we will use to take each of the rows containing the text data and turn them into individual text files.
```{r}
setwd("~/Desktop/QualData")
rbbcsc = read.csv("RBBCSCStaffSurvey.csv", header = TRUE)
rbbcsc = as.data.frame(rbbcsc)
mccsc = read.csv("MCCSCStaffSurvey.csv", header = TRUE)

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
The next step is to create a folder with txt files each with the segment of text that we want analyzed.  For this example, we have answers to an open-ended survey question.  Each answer receives its own txt file.  The user needs to library and source the appropriate packages to use the csv2txt package.  The first argument is the location of an empty folder only containing the data set created in the previous step.  Then the user can use the csv2text package to transform the data set created in the previous step into a set of txt files where each file contains a row of text data.  The first argument for the csv2text package is to identify the location of the folder where you want to place the txt files.  The next argument identifies which column has the labels, which is this case is the id variable, which is the first variable in the data set. 
```{r}
library(devtools)
source_url("https://gist.github.com/benmarwick/9266072/raw/csv2txts.R")
csv2txt("~/Desktop/QualAuto", labels = 1)
```
Now that the text files are created and placed in the appropriate data set, we need to create the control file.  The control file has three columns.  First, is the path / text file location, called filename in this example.  There is likely a cleaner way to accomplish this task, but the code below works.  First, we create a random data frame with the same number of columns as the original data set’s rows.  Then we transpose the data set creating the same number of columns in this data set as the number of rows in the original.  Then we paste the filename locations to the columns and number them from 1 to 401 (the number of responses in this data set) so that the ReadMe package selects each of the txt files, which are names 1.txt through 401.txt.  Finally, we rename the data set as filename so the Readme package knows that this variable is the filename.     
```{r}
set.seed(1)
both1 <- replicate(401, rnorm(1, 0, 1))  
both1 = as.data.frame(t(both1))
dim(both1)
names(both1) <- paste0( "/Users/matthewhanauer/Desktop/QualAuto/",1:ncol(both1), ".txt")
library(reshape2)
both1 = melt(both1)
both1 = both1$variable
both1 = as.data.frame(both1)
colnames(both1) = c("filename")
filename = both1$filename
filename = as.data.frame(filename)
dim(filename)
```
Next, we need to create the other two required columns for the control file truth and training set.  The truth file will be composed of the names or values that you have hand coded.  In this artificial example, we have “hand coded” the data and placed the first 50 values into two categories 1 and 2.  The rest of the values will be left blank, which Readme will automatically code later.  The next column is the training set, which indicates which values are the truth, and will be used to train the Readme algorithm, and which values the test set (i.e. all values not in the training set).  The training set only takes on two values 1 indicating a row is in the training set and a 0 indicating that the data are not in the training set.  Finally, we combine all three columns (filename, truth, and training set) into a variable, which is named "control.txt" and where the row.names are false, so an additional variable is not created and where the file uses a comma , to indicate a new column.  One thing that users may need to be aware is that they may need to get rid of the "" in the written data set.  This is easy to use the replace all function in the txt file by replacing " with nothing.
```{r}
truth1 =  data.frame(a = rep(c(1,2),25))
truth1 = as.data.frame(truth1)
truth2 = data.frame(a = rep("", 401-50))
truth2 = as.data.frame(truth2)
truth = rbind(truth1, truth2)
names(truth) = c("truth")
trainingset1 = data.frame(a = rep(1,50))
trainingset1 = as.data.frame(trainingset1)
trainingset2 = data.frame(a = rep(0, 401-50))
trainingset2 = as.data.frame(trainingset2)
trainingset = rbind(trainingset1, trainingset2)
names(trainingset) = c("trainingset")
control = cbind(filename, truth, trainingset)
write.table(control, "control.txt", row.names = FALSE, sep = ",")
head(control)
```
Finally, we can analyze the data using the ReadMe package, which is very straightforward.  First, we need to set the working directory to the location of the txt and control files.  Then we can use the undergrad function to create a data set with each word written in the files and a binary indicator indicating whether or not the word was written in that survey response.  Then we further process the data by removing the training data set.  Finally, we have the readme function which calculates the percentage of the rows (i.e. the survey responses in this example) that fall into particular categories.  The final result is found in est.CSMF which shows that .44 of the data would fall into category 1 while  .56 of the data would fall into category 2. 
```{r}

library(ReadMe)
setwd("~/Desktop/QualAuto")
undergrad.results = undergrad(sep = ',')

undergrad.preprocess <- preprocess(undergrad.results)
readme.results <- readme(undergrad.preprocess, boot.se = TRUE)
readme.results$est.CSMF
readme.results$CSMF.se
```

