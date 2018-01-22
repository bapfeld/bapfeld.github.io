---
layout: post
title: "Importing Stata and SPSS Files into R"
categories: [Code]
tags: [R]
---
In Stats I this semester, students have to run a preliminary analysis on a question that interests them. This involves finding relevant data, importing it into R, and then running the analysis. I've been helping some students with the second step recently and have found that a lot of them are dealing with a common problem when importing Stata (.dta) or SPSS (.sav) files into R: the variables are renamed and no longer match the codebook. For example, when importing the data, the variables will be converted to something like "V1034" instead of "jud_opin_5". (Not that the latter is terribly informative, just that it appears in the documentation for the data.) The task at hand was to subset the dataset, selecting only the variables of interest.

Here's a quick post on one solution that I've come up with for this problem by recognizing that the variables are being imported with descriptive "attributes." (See [this chapter](http://adv-r.had.co.nz/Data-structures.html) in Hadley Wickham's fantastic Advanced R book if you're interested in learning more about attributes.)

Here's the solution that I came up with:

```r
foo <- rio::import("some_data.dta")
# alternately:
# foo <- rio::import("some_data.sav")
label_value <- c()
for(i in 1:ncol(foo)){
  label_value[i] <- attr(foo[[i]], "label")
}
var_labels <- data.frame(var = names(foo), label = label_value)
```

UPDATE (1/21/18): I've realized that if you have a blank label value in a Stata dataset, instead of writing a blank string or something there, you just end up without that attribute, causing an error if you tried the run the code above. That's an easy fix:

```r
label_value <- c()
for(i in 1:ncol(foo)){
  label_value[i] <- ifelse(!is.null(attr(foo[[i]], "label")), attr(foo[[i]], "label"), NA)
}
var_labels <- data.frame(var = names(foo), label = label_value)
```

After importing the data, we extract the attribute called "label" from each variable. We then create a dataframe with the variable name itself along with the attribute we just extracted. You can search this in two ways. The first is manually (either within R - yuck, or exporting to a csv and using MS Excel) or you could try your hand at `grep()`. As an example, if you wanted to search for variables that contained the word "opinion" in them, you would do `grep("opinion", var_labels$label)`, which would return a vector of the row numbers that contained "opinion". You could also do `grep("opinion", var_label$label, value = TRUE)` to return what those labels actually are, but this is still a little complicated for using that information meaningfully. The best option is probably `var_labels[grep("opinion", var_labels$label),]`, which would return the rows from the `var_labels` dataframe that contain "opinion" along with the number of the row, making it simple to determine which variables you actually want.

If you're not sure if your data has this attribute, run `str(foo)` and it should list any attributes. If you do, it will look something like this:

```r
$ sp40c   : atomic  1 2 1 1 2 2 2 2 2 3 ...
 ..- attr(*, "label")= chr "opinion about european community-japan"
 ..- attr(*, "format.stata")= chr "%8.0g"
 ..- attr(*, "labels")= Named num  0 1 2 3 4 5 8
 .. ..- attr(*, "names")= chr  "na" "very good" "good" "about average" ...
```

 Each attribute is listed below the variable and the name of it is listed as the second argument of `attr()`.

 Of course another option is simply to replace the names of the original variables with these labels:

```r
 for(i in 1:ncol(foo)){
   names(foo)[i] <- attr(foo[[i]], "label")
 }
```

 Depending on what the "label" attribute is in your dataset, this may or may not be useful.

I didn't spend enough time with any of the datasets or their sources to figure out why this was a problem in those cases. Bottom line is that if you find yourself in this position, this may be a quick way to solve the problem.
