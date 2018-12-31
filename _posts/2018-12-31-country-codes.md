---
layout: post
title: "Merging Country Level Observations"
categories: [code]
tags: [R, Python]
---
As a comparative political scientist one of the most frequent data cleaning tasks I face is merging datasets that contain country or country-year observations. It can be a deceptively challenging task. This post is largely based on an email I sent in response to a question about tools to facilitate this kind of merging and will describe an available R package, as well as a couple of tools that I've built (with some help from [Daniel Weitzel](http://danweitzel.github.io) and Ben Chatterton.

# Challenges of Merging Country Level Data 
There are two primary challenges associated with merging country level data. First, country definitions change over time. Borders move; countries are split; territories are acquired. Sometimes this doesn't matter. More often it really matters. Second, at any given point in time, a country may go by more than one name. Or more than one spelling.

Obviously the latter problem only relates to territories that have been identified by some kind of a string, while the former is an issue whether it's a "name" or a code that's being used. 

# The Golden Rule of Merging
What I suggest below is meant to make life a little easier. There's no reason to rewrite tedious code every time you want to merge two datasets. Any approach you use will require some kind of a bridge between the two datasets --- some kind of unique identifier that can be applied to both that allows the merge to happen. This leads to what I'll call the "Golden Rule of Merging", though a better name may be "the rule that you will constantly be tempted to break but will do so only at your own grave peril":

> Know your data backwards and forwards. Know your bridge backwards and forwards.

This rule is applicable to merging any data using any bridge. You must know how the units are defined in your data and in the bridge. You should know what you have and what you expect to get after the merge. Test everything.

# Solutions
Any solution is going to require a bridge of some kind. This could be country name, but I think some kind of a code will always be easier. For any given dataset you translate the identifier into your code, then you can arbitrarily merge datasets based on this code. 

## Countrycode Package in R
If you're dealing exclusively with nicely formatted data that uses preexisting code systems, then the R package [countrycode](https://github.com/vincentarelbundock/countrycode) is a great place to start. It allows you to do a variety of things: convert one standard to another, go from a standard code to a country name, or go from a country to a continent or region. You can even add custom matches if you have some variations outside their "canonical" list. It's very easy to use and is hosted on CRAN, so you can install it simply with `install.packages('countrycode')`. Here's a little bit of code showing how you might use it:

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

I have one major concern about this approach, though. It's not clear what standard (if any) is being used to define countries. As borders change over time, are these units constant or do they vary? If so, how? This limits its usefulness to cases where the data being merged all covers country-years in which borders are constant. The function can then be thought of as acting like a translator between two languages. 

Still, this is a nifty piece of software that's fast, easy to use, and relatively flexible. 

## Vcode Function in R, Stata, or Python
Partly because of the concern above and partly because that package does not (currently) include support for V-Dem country codes (an absolute must for me), I've written an alternative. All code from here on out can be found on its [Github page](https://github.com/bapfeld/vdem-codes). A lot of what follows is a duplication of information available on that page's README. 

The primary function provided here is `vcode` and can be found in the file `vcode_function.R`. It takes a vector of country names and returns corresponding V-Dem codes. The V-Dem Project provides extensive documentation about how they define country units over time.[^fn1] The function's philosophy is fairly simple: maintain an underlying master csv of country name-code pairs; when the function is invoked, the data is loaded and input names are matched against the list; if *any* names are not found in the list, the function returns an error message listing the names that were unmatched so that they can either be fixed in the data, or added to the master list. The list thus grows over time to accommodate new variations as they are encountered, essentially requiring them to be fixed only a single time. 

In order to accommodate this flexibility, or even to allow an entirely new "master" to be used in its place, I have avoided bundling this into a true R package. An additional advantage that comes with this is the flexibility to translate this function into other languages. The files `vcode.py` and `vcode-function.do` provide this in Python and Stata, respectively. 

### Differences Between R, Stata, and Python Functions
The R function is more robust, more flexible, and marginally faster than its Stata counterpart. First, the user can specify a different input CSV file if a particular project needs one that differs from the master in any way. Second, the function allows the user to specify the name of the country variable in the data (e.g. "Country" vs. "country" vs. "Country Name" etc.). Third, the function will not complete if any country names in the input are not found in the CSV file. It will instead return those names so that they can be added to the master file. Fourth, any changes saved to the master CSV file are immediately available for use by the function without any additional actions. Finally, the R function allows the user to specify a custom set of matches if desired. This can be supplied either as a `data.frame` with columns `source` and `destination` or as a named vector in the format `c("origin" = "destination")`.

The Stata .do file is more rudimentary. First, the variable containing country names in the data must be "country". Second, country names that do not appear in the CSV will appear as missing in Stata. Third, and most importantly, changes made to the CSV are not immediately available in the .do file. The tool contains additional code to update the .do file if the master CSV changes, but it requires the user to take additional action (and the user must have both R and GNU Make installed on their machine).

The python function mimics the R function in that it allows the user to specify a country name column, a path to the country-code csv file and issues a warning on missing country names. It has been written to work on a pandas dataframe, but it has not been tested extensively. This function is also currently out of date with its R counterpart and does not offer all of the same functionality. Further, the Python function assumes the user is using a `pandas` DataFrame. The R code has no dependencies, by contrast. 

### Other tools provided in vcode
The vcode repo also contains code to convert a V-Dem code into an English language country name. This is provided as the function `country_from_vcode` found in the `vcode_to_country.R` file. Like `vcode`, it relies on an external master csv that simply links each V-Dem code with a single name. The external csv allows for easy changes if you prefer a different country name variation. Note that this list was originally generated automatically by taking the first name from an alphabetical sort and that mistakes from this are being corrected over time. Double-check before you publish anything with these names! This tool only exists in R.

The repo also contains a Makefile (to be used with GNU Make). It is intended to be run whenever the CSV file is updated. Doing so will update the stata function to match the changes. It also re-sorts the CSV to avoid any duplication. Any entirely new codes are detected and added to a separate file that attaches a single name to each code. The hope is the makefile facilitates keeping everything up to date while avoiding any duplication. 

## Advanced Function in Python
Realizing some of the limitations of the `vcode` function, I've been developing a new approach in Python. The code is very much in a development state (not all of the options it claims to have work, and not all actual options have been fully tested). You can find it in `terr_code_assignment.py`. It relies on the same underlying philosophy of maintaining a master list of country-code pairs, but adds some important flexibility. First, it can be called from the command line, which makes it less accessible but more easily integrated into a full data cleaning pipeline. Second, it provides robust checks on matches, allowing the user to perform a "dry-run" and examine how countries will be coded, as well as options to override warnings if desired. Third, it implements character standardization, reducing the required length of the master list. The `vcode` function, for example, treats "United States" and "united states", or "España" and "Espana" as two separate entities, requiring two entries for each in the master list. The new Python code standardizes all names before trying to match. Finally, the code allows for flexibility in using either csv of xlsx files. 

This code is definitely still in development, but it's been run successfully on both Windows and Mac machines and will produce output from both the dry-run and main functions. I believe this is the future direction of this project and hope to bring some of this functionality back to the R code. 

I plan to make this into a fully-functional (and standalone) python module soon (along with a matching R version). If you're interested in trying this out now, here's how you would do it:

### The function
```bash
terr_code_assignment.py [-h] --file-path FILE_PATH --territory-column
                        TERRITORY_COLUMN --territory-code-file
                        TERRITORY_CODE_FILE
                        [--territory-id-column TERRITORY_ID_COLUMN]
                        [--custom-matches CUSTOM_MATCHES] --dry-run
                        DRY_RUN [--dry-run-out DRY_RUN_OUT]
```

The arguments are:
- `--file-path`: the path to the file you want to use. It can be either a csv or an xlsx file.
- `--territory-column`: the name of the column in the data that contains the territory name to be coded. Column names with spaces or weird characters can be placed in back-ticks.
- `--territory-code-file`: the path to the master territory code file (`territory_list.csv`)
- `--territory-id-column`: the name for the new territory code variable. Defaults to `territory_id` and is not required
- `--custom-matches`: currently not used
- `--dry-run`: a boolean flag indicating if this is a dry-run or not
- `--dry-run-out`: the path to where you want to write the dry-run file. Can be either a csv or an xlsx file.

### Workflow
I imagine that the workflow for using this proceeds as follows. I'm going to assume you are already in the directory that contains the data you want to work on and that you have created the alias `terr_code` for calling the script.

1. Dry-run
 First, we want to see what is going to match up:
 
 ```bash
 terr_code --file-path=my_data.xlsx \\
          --territory-column=country \\
          --territory-code-file=territory_list.csv \\
          --dry-run=True \\
          --dry-run-out=test_matches.xlsx
 ```
 
 This will produce an excel file with unique territory (country) names from the "my_data.xlsx" file, the cleaned up version of that name, and the corresponding territory code (if it exists).

2. Examine and fix
 This is the time consuming step. Here you'll have to go through the list and make sure that everything that's been matched and make decisions about whether or not it's correct. For correct matches, nothing needs to be done. For incorrect matches, you'll have to make changes to the input file. For non-matches, you can either make changes to the input file or the territory list file, depending on what's required. If it's a truly new territory, then the only solution is to add a new territory/code pair to the territory list file. If it's an alternative spelling/name, then you have an option about which file to edit. I would recommend editing the territory list file in this case only if you are certain that doing so won't create problems in the future with other territory names. 
 
3. The "real" run
 Finally, we rerun `terr_code` and have it write the codes to the original data file.
 
 ```bash
 terr_code --file-path=my_data.xlsx \\
          --territory-column=country \\
          --territory-code-file=territory_list.csv \\
          --territory-id-column=territory_id \\
          --dry-run=False
 ```
 
 This will assign the codes and write the file back to the file path you specified with `--file-path`.
 
### Known Pitfalls
- During the real assignment of codes, there is no warning if a name doesn't match so it is possible to still end up with blank territory codes.
- The `custom_matches` option is not currently setup to work 
- I've done some error checking here, but I have not thoroughly tested this, so it's possible that it will throw unhelpful python errors instead of doing something more informative
- Similarly, this does some error checking on inputs, but there are probably some ways to make it break that I haven't thought of
- If you are migrating from the vcode function, then be aware that this will run even with missing country names, though it should warn in that case

## Code snipped specifically for Gleditsch and Ward Codes
Finally, if you ever need to merge data from Gleditsch and Ward's research (available on Kristian Gleditsch's [website](http://ksgleditsch.com/data.html)), here's a code snippet that will make your life easier. Their coding generally follows coding from the Correlates of War project, but there are a few minor differences, so this snippet will go from their codes to readable names.

```R
# Download the G&W Codes
gw <- data.table::fread("http://ksgleditsch.com/data/iisystem.dat", encoding = "Latin-1")
gw_micro <- data.table::fread("http://ksgleditsch.com/data/microstatessystem.dat", encoding = "Latin-1")
gw <- rbind(gw, gw_micro)
gw <- rbind(gw, t(c(679, "YEM", "Yemen", "", "")))
gw$V1 <- as.numeric(gw$V1)

# Assuming a dataframe, df, with the GW code as variable gw_code:
df$country <- gw$V3[match(df$gw_code, gw$V1n)]
```

[^fn1]: As of this post, the current version of V-Dem data and corresponding literature is Version 8. It, and country-unit definitions are available at [https://www.v-dem.net/en/reference/version-8-apr-2018/](https://www.v-dem.net/en/reference/version-8-apr-2018/).
