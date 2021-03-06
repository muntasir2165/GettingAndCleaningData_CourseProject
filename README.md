Getting and Cleaning Data Course Project Walkthrough
=====================================================

Set the working directory to be the Course Project folder

```r
setwd("~/Desktop/Online/OWinter Term 2014/Getting and Cleaning Data/Course Project")
```

TRAINING DATA:
Read in the training data into R

```r
X_train <- read.table("./UCI HAR Dataset/train/X_train.txt")
y_train <- read.table("./UCI HAR Dataset/train/y_train.txt")
subject_train <- read.table("./UCI HAR Dataset/train/subject_train.txt")
```

TEST DATA
Read in the test data into R

```r
X_test <- read.table("./UCI HAR Dataset/test/X_test.txt")
y_test <- read.table("./UCI HAR Dataset/test/y_test.txt")
subject_test <- read.table("./UCI HAR Dataset/test/subject_test.txt")
```

**1. Merge the training and the test sets to create one data set.**

Merge the training and test X, y, subject data together

```r
X <- rbind(X_train, X_test)
y <- rbind(y_train, y_test)
subject <- rbind(subject_train, subject_test)
```

Concatenate the X, y, and subject data into a data frame

```r
data <- data.frame(subject, y, X)
```

**2. Extract only the measurements on the mean and standard deviation for each measurement.**  
**4. Appropriately labels the data set with descriptive variable names.**

Read in the features.txt into R that contains the 561 feature names

```r
features <- read.table("./UCI HAR Dataset/features.txt")
class(features[,1]) # => integer
class(features[,2]) # => factor
features[,2] <- as.character(features[,2])
class(features[,2]) # => character
```

Replace the '-', ',', '(', ')', characters in the features character
vector with '_', "", "", and "" respectively to create valid R variable
names

```r
features[,2] <- gsub("-", "_", features[,2])
features[,2] <- gsub(",", "", features[,2])
features[,2] <- gsub(")", "", features[,2])
features[,2] <- gsub("\\(", "", features[,2])
```

Initialize a character vector to store the features involving mean and standard deviation data for subsetting, processing, and data frame naming later on

```r
featureNames <- vector(mode="character")
```

Initialize a numeric vector to store the index of the features involving mean and standard deviation for data processing later on

```r
featureColumns <- vector(mode="numeric")
```

Extract the names and column indexes of the measurements of means and standard deviations for creating a new data frame

```r
#create an indexing variable
i <- 0

for (name in features[,2]) {
        i <- i + 1
        #print(i)
        meanMatch = grepl("mean", name)
        sdMatch = grepl("std", name)
        if (meanMatch || sdMatch) {
                featureNames <- append(featureNames, name)
                featureColumns <- append(featureColumns, i)
        }
}
```

Create a new data frame with subjectID, activity numbers and the columns pertaining to measurements of mean and standard deviation

```r
dataset <- cbind(subject, y, X[, featureColumns])
```

Give names to the columns in the dataset data frame

```r
names(dataset) <- c("subjectID", "activity", featureNames)
```

**3. Uses descriptive activity names to name the activities in the data set**

Read in the activity_labels.txt into R that contains the numeric labels for the 6 activities

```r
activityFile <- read.table("./UCI HAR Dataset/activity_labels.txt")
activity <- activityFile
class(activity[,1]) # => integer
class(activity[,2]) # => factor
activity[,2] <- as.character(activity[,2])
class(activity[,2]) # => character
```

Replace the integers 1, 2, 3, 4, 5, 6 in the y column of data with the corresponding activity label names

```r
for (i in seq(activity[,1])) {
        y[,1] <- gsub(activity[i,1], activity[i,2], y[,1])   
}
```

Modify the dataset data frame with the updated y column containing name-based labels

```r
dataset[,2] <- y
```

**5. Create a second, independent tidy data set with the average of each variable for each activity and each subject.** 

Initialize the columns of the tidy data frame

```r
subjectID <- rep(1:30, 1, each=6)
activityVector <- rep(activity[,2], 30)
measurement <- matrix(rep(0, (ncol(dataset)-2)*180), nrow=180)
tidy <- data.frame(subjectID, activityVector, measurement)
```

Name the columns of the tidy data frame

```r
names(tidy) <- c("subjectID", "activity", featureNames)
```

Populate the tidy data set with the average of each variable for each
activity and each subject. 

```r
for (row in seq(subjectID)) {
        subject <- subjectID[row]
        #print(paste(row, subject, sep=": "))
        for (act in activity[,2]) {
                measurementColumns <- dataset[dataset$subjectID==subject & dataset$activity==act, 3:ncol(dataset)]
                currentMeasurementMean <- apply(measurementColumns, 2, mean)
                tidy[row,3:ncol(tidy)] <- currentMeasurementMean
        }
}
```

Save the tidy data frame containing the aggregate data in a .txt file

```r
write.table(tidy, file='tidyDataTXT.txt')
```

Additionally, save the tidy data frame containing the aggregate data in
a .csv file

```r
write.csv(tidy, file='tidyDataCSV.txt')
```

Save the featureNames vetor containing the measurement meand and standard deviation features names in our tidy data into a .txt file. Then, we can conveniently copy these names from the .txt file into our code book

```r
write.table(featureNames, file='featureNames.txt')
```
