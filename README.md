if(!file.exists("./final")){  dir.create("./final")}

fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"download.file(fileUrl,destfile="./final/SPhoneData.zip")

## Make sure that all unzipped files are moved to a single folder within the working directory.  In total, 28 files are extracted; some of which consist of redundant data.  For data-cleansing purposes, we are concerned with merging the following files:

## 1)X_test, 2) X_train, 3) Features, 4) y_test, 5) y_train, 6) subject_test, 7) subject_train

## The above 7 files will be clean, transformed and combined into a master datafram ("MasterDF"), which itself will be manipulated and subsetted to produce the ("tidyDF2") dataframe consisting of averages of each variable for each activity and each subject.

## Data for the full sample (30 people/subjects) is partitioned across## the "test" group (9 subjects) and the "train" group (21 subjects). 

## Both the "test and "train" groups collect data for the same 561 variables## (as referenced in the "features" file).  

X_test <- read.table ("./final/X_test.txt")
X_train <- read.table ("./final/X_train.txt")
features <- read.table ("./final/features.txt")

## The "Y" files for both "test" and "train" contain information relating to## activity; refrencing the "activities file, codes are as follows: 1) Walking, 2) Walking_Upstairs, 3) Walking_Downstairs, 4) Sitting, 5) Standing, 6) Laying

y_test <- read.table ("./final/y_test.txt")y_train <- read.table ("./final/y_train.txt")

## Make "y_test" and "y_train" more understandable by substuting the indexes 1:6 with the actual, descriptive names; to preserve the orignal data, intermediate variable "y_test2" created

y_test2 <- y_testy_test2$V1 <- factor(y_test2$V1, levels = c(1,2,3,4,5,6), labels = c("WALK", "WALK_UP", "WALK_DOWN", "SIT", "STAND", "LAY"))

## To preserve the orignal data, intermediate variable "y_train2" created

y_train2 <- y_trainy_train2$V1 <- factor(y_train2$V1, levels = c(1,2,3,4,5,6), labels = c("WALK", "WALK_UP", "WALK_DOWN", "SIT", "STAND", "LAY"))

## Rename the columns labels from the default "V1" to "Activity":

colnames(y_test2) <- c("Activity")colnames(y_train2) <- c("Activity")
y_combined <- rbind(y_test2, y_train2)

## Apply the same procedures for the "subject" files as we did for the "y" files

subject_test <- read.table ("./final/subject_test.txt")subject_train <- read.table ("./final/subject_train.txt")
subject_test2 <- subject_testsubject_train2 <- subject_train
colnames(subject_test2) <- c("PersonNum")colnames(subject_train2) <- c("PersonNum")
subject_combined <- rbind(subject_test2, subject_train2)

## Merge xtest and xtrain. X_combined2 created to preserve orignal data

X_combined <- rbind(X_test, X_train)
X_combined2 <- X_combinedX_combined2 <- setNames(X_combined2, features2)

## Original "features" variables are columnar; we need to transpose "features"## to horizontal vector so that it can be attached to X_combined as a label vector:

features2 <- t(features[2])

## Subset to include only mean and stdev data (per project instructions); this step##  must be performed prior to the cbind step in order to avoid offset errors

X_combined3 <- X_combined2[,c(1:6, 41:46, 81:86, 121:126, 161:166, 201:202, 214:215, 227:228, 240:241, 253:254, 266:271, 294:296, 345:350, 373:375, 424:429, 452:454, 503:504, 513, 516:517, 526, 529:530, 539, 542:543, 552, 555:561)]

## (Optional) Check dimensions of final 3 building blocks

dim(subject_combined)
dim(y_combined)
dim(X_combined3)

## Create MasterDF:

MasterDF <- cbind(subject_combined, y_combined, X_combined3)
dim(MasterDF)

## Dataframe consisting of averages of each variable for each activity and each subject head(MasterDF)

library(reshape2)
tidyDF1 <- melt(MasterDF, id = c("PersonNum", "Activity"))
tidyDF2 <- dcast(tidyDF1, Activity + PersonNum ~ variable, mean)
