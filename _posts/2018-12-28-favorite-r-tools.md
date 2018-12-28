---
layout: post
title: "Favorite R Code"
categories: [Code]
tags: [R, Workflow]
---
I also find that the longer I use R, the more my preferred solutions converge around a decreasing number of commands. But every now and then I find something new that makes life so much easier and it finds its way into my repertoire. Here's a quick post on some of these (code) life-changing commands. Some of these are fairly well-known, others less-so. 

# Base R functions
Here are a few that come from base R (and can be applied in any situation)

1. `%in%`
I don't think I could live without `%in%`. As its name implies, it checks if an object or a value is present in some other object. So if you have a list `fruit <- c('apples', 'oranges', 'bananas')` and you want to know if `pears` is in your list: `'pears' %in% fruit` will return a logical `FALSE`. This has numerous applications, but I probably use it most frequently for subsetting dataframes. So let's take a toy example where you have some data that has observations for all US States, but you only want the ones for Texas and Wisconsin (obviously).

```R
dat <- read.csv('my_amazing_data.csv', header = T, sep = ",", stringsAsFactors = F)

# The bad old way
dat <- dat[which(dat$state == "Texas" | USarrests$state == "Wisconsin"),]

# The better new way
dat <- dat[which(dat$state %in% c("Texas", "Wisconsin"),]
```

Even when we only have a list of two things to test membership against, our code is shortened. The advantage obviously grows even more as that list grows. 

2. `match`
This is closely related to `%in%` but serves a slightly different purpose. According to the help file, 

> `match` returns a vector of the positions of (first) matches of its first argument in its second.

Using the fruit example again, `match('oranges', fruit)` will return `2`, indicating that `'oranges'` is the second element in `fruit`. Note that if you tried `match('pears', fruit)`, R would return `NA`. I find this to be a really intuitive way to assign values from one dataframe to another where some values may be repeated. 

3. `setdiff`, `union`, and `intersect`
Logical operators for sets that can make life so much easier.[^1] Fans of Venn diagrams will love these. They act logically according to their names, can be strung together, and are highly optimized. 

4. `get` and `mget`
These two commands (a single and multiple version, respectively) will return an actual object from a quoted string. Let's say you have a series of dataframes `a1` through `a4` and you need to create a new object that is a list of those four dataframes that you'll pass on to some other function. If you just run `df_list <- c('a1', 'a2', 'a3', 'a4')`, you'll end up with a list of four strings, not the dataframes themselves. You could solve this by changing your syntax to `df_list <- mget('a1', 'a2', 'a3', 'a4')`. And if you had a really long list of objects, say `a1` through `a999`, then you could even do something like this: `df_list <- mget(grep('a\\d+', ls(), value = T))`. 

This is often useful within custom functions. For example, if you have a function that takes a string argument but references an actual object. If you're using the tidyverse you'll have to solve this problem in other ways (read the dplyr documentation on [quotation and quasiquotation](https://dplyr.tidyverse.org/articles/programming.html#quoting).

5. `assign`
This is the full command version of the assignment arrow `<-`. The order of the arguments is the same, read left to right, but you need to quote the name you want to use. So `my_var <- 5` is equivalent to `assign('my_var', 5)`. 

Like `get`, this can be very useful within custom functions for two reasons.[^2] First, it allows you to create a function that takes a string to use as a name for an object created within the function. Second, the `assign` function has an additional argument `envir` that specifies the environment in which to create the object. Normally everything happening within a function happens within its own environment, so any variables that you create within the function will disappear when it completes. `assign` can overcome this and create in the global environment. Compare these two functions:

```r
func_1 <- function(x){
    draws <- rnorm(x)
}
func_1(10)
print(draws)
# Will return "Error in print(draws) : object 'draws' not found

func_2 <- function(x){
    assign("draws", rnorm(x), envir = .GlobalEnv)
}
func_2(10)
print(draws)
# Will return the vector of values
```


6. `do.call`
From the documentation,

> ‘do.call’ constructs and executes a function call from a name or a function and a list of arguments to be passed to it.

The way I use this (and the only way I've seen others use it) is to repeat a function multiple times. In particular, it's great for collapsing a list. The same thing can be accomplished with `purrr::reduce`. The advantage of `do.call` is that it's base R (and you don't have to try to remember how many Rs are in `purrr`). 

Here's a (mostly) real example from a project I'm working on. Note: don't try to run this unless you want your computer to be busy for a very long time. The code defines a function then replicates it 100,000 times using parallel execution. The result is a list that needs to be flattened into a data.frame, which happens in the final line.

```r
rnd_experiment <- function(i){
  samp_data_a <- sample(nums, 150000, replace = TRUE)
  samp_data_b <- sample(nums, 150000, replace = TRUE)
  samp_data_c <- sample(nums, 150000, replace = TRUE)
  samp_dat <- as.data.frame(cbind(samp_data_a,
                                  samp_data_b,
                                  samp_data_c))
  samp_dat$score <- apply(samp_dat, 1, mean)
  rd_test <- rddensity::rddensity(samp_dat$score, c = 15.7)
  out_row <- c(t = rd_test$test$t_jk, p = rd_test$test$p_jk)
  return(out_row)
}

tests <- parallel::mclapply(1:1e5,
                            rnd_experiment,
                            mc.cores = (parallel::detectCores() - 1))
tests <- as.data.frame(do.call("rbind", tests))
```

# Tidyverse related
These either come from the tidyverse and/or are used exclusively with those packages

7. `%>%` and `%<>%`
I'm trying to move away from the tidyverse world, but that's hard to do partly because of the amazing convenience of pipes. The simple forward pipe `%>%` is imported by `dplyr` these days, but it and its other pipe cousins actually come from the `magrittr` package. These pipes let you string together a series of transformations in a really easy and logical way.

```R
dat <- dat %>%
    filter(state %in% c("Texas", "Wisconsin") %>%
    select(state, year, other_column)
```

In short, this says "dat gets dat, THEN filter observations THEN select certain variables." Easy! But `dat <- dat` is ugly. So let's use the compound assignment pipe:

```R
dat %<>%
    filter(state %in% c("Texas", "Wisconsin") %>%
    select(state, year, other_column)
```

That's better. `magrittr` has several other pipes (take a look [here](https://cran.r-project.org/web/packages/magrittr/vignettes/magrittr.html), but these are the only two that are in my current workflow. 

8. `do`
For all those times when you want to combine a custom function you wrote (or even a base function that doesn't have a tidy equivalent) with a dplyr-esque pipe. I find this really helpful when combined with `group_by`.

# Other packages
And finally a few that come from other packages (but still work anywhere)

9. `wrapr::qc()`
Very basic function that takes unquoted names and returns a quoted list. 

```R
# old way
fruits <- c('apples', 'oranges', 'bananas')
fruits

# [1] "apples"  "oranges" "bananas"

# new way
fruits <- qc(apples, oranges, bananas)
fruits
# [1] "apples"  "oranges" "bananas"
```

Okay, so for this example that only saves 5 keystrokes, but for longer lists it can really come in handy. One downside is that I don't really want to load the entire `wrapr` library and R doesn't have a nice Python-esque `from wrapr import qc` option[^3], so I'm forced to add `qc <- wrapr::qc` to the start of my scripts.

10. `countrycode::countrycode()`
This is an enormously useful function for converting country codes from one standard to another for those times when you want to merge two datasets with country level observations and no shared identifier. This is a problem I face ALL THE TIME and will write a full post on it soon, but here's a quick overview of what you can do with this one function.

```R
# Go from one standard to another, including a custom match
cow$c1 <- countrycode::countrycode(cow$ccode1,
                                   origin = "cown",
                                   destination = "country.name",
                                   custom_match = c("710.1" = "Hong Kong"))
# Go from a code to a readable name
gn$country_name <- countrycode::countrycode(gn$country_code,
                                            origin = "iso2c",
                                            destination = "country.name")

# Go from country to continent
continents <- countrycode::countrycode(my_countries,
                                       origin = "country.name",
                                       destination = "continent")
```

[^1]: "Set," of course, has no special meaning in R, unlike other languages. I'm using it here to refer to any vector.
[^3]: As an aside, the documentation actually notes that `assign` and `get` are inverse functions.
[^3]: At least it doesn't as far as I'm aware. Please disabuse me of this idea if such a solution exists.

