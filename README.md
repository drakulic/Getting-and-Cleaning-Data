Getting and Cleaning Data Project by George Drakulic
========================================================

## download data from
## http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

## unzip it at your working directory
## it will create directoy called "UCI HAR Dataset"
## in that directory there are two other directories: "test" and "train",
## as well as labels of variour columns of the data in txt files

## create a data frame "subject" by reading subject_test.txt, which has rows of a subject's number,
## and name the column "subject"

```{r}
subject <- read.table("UCI HAR Dataset/test/subject_test.txt")
colnames(subject) <- c("subject")
```

## create a data frame "activity" by reading y_test.txt, which has rows of an activity's number,
## and name the column "activity"

```{r}
activity <- read.table("UCI HAR Dataset/test/y_test.txt")
colnames(activity) <- c("activity")

```

## change the numbers in activity data frame into textual activities listed in "activity_labels.txt"

```{r}
activity <- factor(activity$activity, levels = c(1, 2, 3, 4, 5, 6), labels = c("walking", "walking_upstairs", "walking_downstairs", "sitting", "standing", "laying"))
```

## merge two data frames into one (subject and activity into dat1)

```{r}
dat1 <- cbind(subject, activity)
```

## read X_test data with 561 columns

```{r}
X_test <- read.table("UCI HAR Dataset/test/X_test.txt")
```

## get the names of the features (which will be column names of X_test data)

```{r}
features <- read.table("UCI HAR Dataset/features.txt")
```

# rows should be columns, so we will transpose rows and columns, and get just the names of the features

```{r}
feature_name <- t(features)[2,]
```

## rename the columns of X_test data with the column names of the features from features.txt

```{r}
colnames(X_test) <- feature_name
```

## get the index of columns that have "mean()" in them, and also "meanFreq()

```{r}
g_mean <- grep("mean()", feature_name)
```

## get the index of columns that have "std()" in them

```{r}
g_std <- grep("std()", feature_name)
```

## combine those indices into one

```{r}
g_all <- append(g_mean, g_std)
```

## sort the indices
## these are the indices we need in our data

```{r}
sorted_indices <- sort(g_all)
```

## the data set with columns of needed data of mean, and std computations

```{r}
X_test_mean_and_std <- X_test[, sorted_indices]
```

## merge subject + activity (dat1), and X_test_mean_std

```{r}
X_test_merged <- cbind(dat1, X_test_mean_and_std)
```

## just to check the final result: nrow(X_test_merged) gives me 2947,
## ncol(X_test_merged) gives me 81, so the test merged data frame is 2947 x 81

## Do the same for the train data

```{r}
subjectTrain <- read.table("UCI HAR Dataset/train/subject_train.txt")
colnames(subjectTrain) <- c("subject")

activityTrain <- read.table("UCI HAR Dataset/train/y_train.txt")
colnames(activityTrain) <- c("activity")

activityTrain <- factor(activityTrain$activity, levels = c(1, 2, 3, 4, 5, 6), labels = c("walking", "walking_upstairs", "walking_downstairs", "sitting", "standing", "laying"))
dat2 <- cbind(subjectTrain, activityTrain)
colnames(dat2) <- c("subject", "activity")

X_train <- read.table("UCI HAR Dataset/train/X_train.txt")
colnames(X_train) <- feature_name
X_train_mean_and_std <- X_train[, sorted_indices]

X_train_merged <- cbind(dat2, X_train_mean_and_std)
```

## just to check the final result: nrow(X_train_merged) gives me 7352,
## ncol(X_train_merged) gives me 81, so the train merged data frame is 7352 x 81

## merging the test and the training sets should give me (2947 + 7352) x 81 data frame
## i.e. 10299 x 81

```{r}
X_final <- rbind(X_test_merged, X_train_merged)
```

## nrow(X_final) gives me 10299, which is as expected, ncol(X_final) is 81 as well

## empty data frame for tidy data

```{r}
X_tidy <- data.frame()
```

# the data is split by subject, activity, and other variables, so it computes
# the mean of each variable for each subject and his/her activity,
# e.g. subject 1, activity 1, all means of variables; subject 2, activity 1, means,
# up to last subject, activity 1
# then, it starts with subject 1, activity 2 ..., last s, act 2 and so on

```{r}
data_split <- split(X_final[, 3:81], list(X_final$subject, X_final$activity))
X_tidy <- sapply(data_split, colMeans)
```

# nrow(X_tidy) = 79, ncol(X_tidy) = 180

# output with row.names=F so the data is properly aligned in MS Excel, but the row names
# are, of course, missing

```{r}
write.table(X_tidy, file="./X_tidy.txt", sep="\t", row.names = FALSE)
```

# output with row.names=T so the data is NOT properly aligned in MS Excel

```{r}
write.table(X_tidy, file="./X_tidy_with_row_names.txt", sep="\t")
```

```{r}
head(X_tidy, 2)
```

