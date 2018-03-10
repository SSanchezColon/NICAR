Creating functions and packages
========================================================
author: Andrew Ba Tran
date: 3/10/2018
autosize: true

Sisyphus
========================================================

Find yourself repeating the same tasks over and over again?

**Example:**

- Analyzing data that's published regularly
- Always replacing that one county name so it matches a lookup file
- Adjusting summary state data for population.

Reproducibility
========================================================

## Save your most common tasks as functions

<img src="https://cdn.shopify.com/s/files/1/0535/6917/products/sisyphus-shirt.gif?v=1414446009" width="100%">


Example: Data with counts by state
========================================================

Open `rmd/04-function_example.Rmd` and run the chunks when spotted

<img src="images/sbcount.png">

```r
sb <- read.csv("https://docs.google.com/spreadsheets/d/1gH6eUQVQsEmFagy0qzQDEuwb3cutMWaddCLc7ESbzjc/pub?gid=294374511&single=true&output=csv", stringsAsFactors=F)
```

Bring in lookup file
========================================================

1. Start with a look up file

(You can bring in a Google Sheet if you publish as a CSV and copy the link over)

<img src="images/lookup.png">

Bring in lookup file
========================================================


```r
pop <- read.csv("https://docs.google.com/spreadsheets/d/16oW_uvRJCNoOnCeAkJH4fDouFokjaGUdGFUCaFdKd6I/pub?output=csv", stringsAsFactors=F)
```

Figure out the columns to join by
========================================================
Which ones match? 

<img src="images/columns.png">

Figure out the columns to join by
========================================================


```r
library(tidyverse)
sb_adjusted <- left_join(sb, pop, by=c("State_Abbreviation"="Abbrev"))

library(knitr)
kable(head(sb_adjusted, 3))
```



|State_Abbreviation | Starbucks|State    | Population|
|:------------------|---------:|:--------|----------:|
|AK                 |        42|Alaska   |     741894|
|AL                 |        65|Alabama  |    4863300|
|AR                 |        37|Arkansas |    6931071|


```r
sb_adjusted$per_capita <- sb_adjusted$Starbucks/sb_adjusted$Population*100000
kable(head(sb_adjusted, 3))
```



|State_Abbreviation | Starbucks|State    | Population| per_capita|
|:------------------|---------:|:--------|----------:|----------:|
|AK                 |        42|Alaska   |     741894|   5.661186|
|AL                 |        65|Alabama  |    4863300|   1.336541|
|AR                 |        37|Arkansas |    6931071|   0.533828|

Adjust the code for general data sets
========================================================
Establish some rules.

All data sets you want to join with the population data set:

1. First column will contain either the full state name or the state abbreviation
2. Second column will have values you want to adjust for population

Adjust the code for general data sets
========================================================


```r
# Save the dataframe as a consistent name
any_df <- sb

# Rename the first column to "Abbrev"
colnames(any_df)[1] <- "Abbrev"

# Join by the similar name
df_adjusted <- left_join(any_df, pop, by="Abbrev")

# Do the calculations based on the values in the second column
df_adjusted$per_capita <- df_adjusted[,2] / df_adjusted$Population * 100000

kable(head(df_adjusted, 3))
```



|Abbrev | Starbucks|State    | Population| per_capita|
|:------|---------:|:--------|----------:|----------:|
|AK     |        42|Alaska   |     741894|   5.661186|
|AL     |        65|Alabama  |    4863300|   1.336541|
|AR     |        37|Arkansas |    6931071|   0.533828|

Turn code into a function
========================================================

Turn your lines of code into a function by wrapping it with 

`function(arg1, arg2, ... ){` and `}`

Remember how there were two types of State ID data? Full name and abbreviations. We can write the function so you can tell it to join based on what type it should join by.

```r
pc_adjust <- function(any_df, state_type){
  pop <- read.csv("https://docs.google.com/spreadsheets/d/16oW_uvRJCNoOnCeAkJH4fDouFokjaGUdGFUCaFdKd6I/pub?output=csv", stringsAsFactors=F)
  # State type options are either "Abbrev" or "State"
colnames(any_df)[1] <- state_type
df_adjusted <- left_join(any_df, pop, by=state_type)
df_adjusted$per_capita <- df_adjusted[,2] / df_adjusted$Population * 1000000
return(df_adjusted)
}
```


Test it out
========================================================

```r
kable(head(sb, 3))
```



|State_Abbreviation | Starbucks|
|:------------------|---------:|
|AK                 |        42|
|AL                 |        65|
|AR                 |        37|


```r
test <- pc_adjust(sb, "Abbrev")
kable(head(test, 3))
```



|Abbrev | Starbucks|State    | Population| per_capita|
|:------|---------:|:--------|----------:|----------:|
|AK     |        42|Alaska   |     741894|   56.61186|
|AL     |        65|Alabama  |    4863300|   13.36541|
|AR     |        37|Arkansas |    6931071|    5.33828|

Try it on another data set
========================================================
Alright, we've got it working with Starbucks data.

Let's try it with Dunkin' Donuts data.


```r
dd <- read.csv("https://docs.google.com/spreadsheets/d/1TWuWZpfDUMWmMpc7aPqUQ-g1a1J0rUO8_cle_zcPyI8/pub?gid=1983903926&single=true&output=csv", stringsAsFactors=F)

kable(head(dd))
```



|State      | Dunkin|
|:----------|------:|
|Alabama    |     18|
|Alaska     |      0|
|Arizona    |     59|
|Arkansas   |      7|
|California |      2|
|Colorado   |      8|

Try it on another data set
========================================================
The state identification is spelled out this time and not abbreviations.

Fortunately, we accounted for that when making the formula.



```r
dd_adjusted <- pc_adjust(dd, "State")

kable(head(dd_adjusted))
```



|State      | Dunkin|Abbrev | Population| per_capita|
|:----------|------:|:------|----------:|----------:|
|Alabama    |     18|AL     |    4863300|  3.7011905|
|Alaska     |      0|AK     |     741894|  0.0000000|
|Arizona    |     59|AZ     |    2988248| 19.7440105|
|Arkansas   |      7|AR     |    6931071|  1.0099449|
|California |      2|CA     |   39250017|  0.0509554|
|Colorado   |      8|CO     |    5540545|  1.4439013|

Yay, we did it!
========================================================

```
pc_adjust() is your tiny perfect function
```

<img src="https://media.giphy.com/media/26xBABFAnPoa4BZMk/source.gif" width="100%">

Keep going!
========================================================

### Save your function as a package for everyone to use
<img src="images/good_time.png" width="100%">


Create a new package
========================================================
**File > New Project > New Directory > R Package**

<img src="images/pckg1.png" width="100%">

Name the package
========================================================
One word. Some [tips](http://r-pkgs.had.co.nz/package.html) on figuring out the best name.

<img src="images/pckg2.png" width="100%">

Three components
========================================================

* An `R/` folder where you save your function code - [more details](http://r-pkgs.had.co.nz/r.html#r)
* A basic `DESCRIPTION` file for package metadata - [more details](http://r-pkgs.had.co.nz/description.html#description)
* A basic `NAMESPACE` file, which is only necessary if you're submitting to CRAN - [more details](http://r-pkgs.had.co.nz/namespace.html#namespace)

<img src="images/pckg4.png" width="100%">


Welcome script
========================================================

<img src="images/pckg3.png" width="100%">


Edit the DESCRIPTION file
========================================================

<img src="images/desc.png" width="100%">

Edit the DESCRIPTION file
========================================================

Questions about which License to use? Check out [the options](http://r-pkgs.had.co.nz/description.html#license).

Also, notice that I added `Imports: dplyr` because this function won't work without the `left_join` function from **dplyr**.
<img src="images/details.png" width="100%">


Create a new script
========================================================
Copy and paste the `pc_adjust` function you made into a new script file.

```
pc_adjust <- function(any_df, state_type){
  pop <- read.csv("https://docs.google.com/spreadsheets/d/16oW_uvRJCNoOnCeAkJH4fDouFokjaGUdGFUCaFdKd6I/pub?output=csv", stringsAsFactors=F)
  # State type options are either "Abbrev" or "State"
colnames(any_df)[1] <- state_type
df_adjusted <- left_join(any_df, pop, by=state_type)
df_adjusted$per_capita <- df_adjusted[,2] / df_adjusted$Population * 1000000
return(df_adjusted)
}
```

Save it as new script in the R folder
========================================================

Name the file after the function, `pc_adjust` and save it into the `R/` folder

<img src="images/new_script.png" width="100%">


Add documentation to your script
========================================================

Go back to your `pc_adjust.R` script and add these lines above the code.

```
#' Population adjuster
#'
#' This function appends state population data
#' @param any_df The name of the dataframe you want to append to
#' @param state_type if state identification is abbreviations, use "Abbrev" if full state name, use "State"
#' @keywords per capita
#' @import dplyr
#' @export
#' @examples
#' pc_adjust(dataframe, "Abbrev")

```

What is all that gibberish?
========================================================

These special comments above the function will be compiled into the correct format.

Watch.

Run these lines in console.

```
install.packages("roxygen2")
library(roxygen2)
roxygenise()
```

Code in the console
========================================================
It wrote to the `NAMESPACE` file and created a `pc_adjust.Rd` file based on the special comments.

<img src="images/roxy.png" width="100%">

Find and open pc_adjust.Rd in the man folder
========================================================

This would've been tough to put together by hand
<img src="images/roxygenise.png" width="100%">


Build your package
========================================================

Press `Cmd + Shift + B` to build the package.

You have your package forever and ever
========================================================

Just run
```
install.packages("whateveryoucalledyourpackage")
```
and you can run `pc_adjust` whenever you want.

Your help file
========================================================

Type
```
?pc_adjust
```
<img src="images/helpfile.png" width="100%">

This is what your special comments above your R function helped generate.

Hold up
========================================================

<img src="http://www.rockpapercynic.com/images/but-wait-theres-more.gif" width="100%">

Upload your package folder to Github
========================================================

<img src="images/github.png" width="100%">


Let others download and use your R package
========================================================

This means you have to add some clean documentation, such as a `readme.MD` file.

```
install.packages("devtools")
library(devtools)

install_github("andrewbtran/abtnicarr")
library(abtnicarr)
```

Better resources
========================================================
From [Giora Simchoni:](http://giorasimchoni.com/rstudio_conf_2018_talk.html#/20)

- Hilary Parker: [Writing an R package from scratch](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/) | ~5 min
- Karl Broman: [R package primer](http://kbroman.org/pkg_primer/) | ~1 hour
- Hadley Wickham: [R packages](http://r-pkgs.had.co.nz/) | ~A few days
- Stack Overflow: [Stack Overflow](https://stackoverflow.com/questions/tagged/r-package) |~Lifetime


Next steps
========================================================

Keep adding functions to your package.

Perhaps, create a [Shiny version](http://shiny.trendct.org/ctnamecleaner/) of it for those who don't use R.

Over time you'll build up a bunch that you'll rely on over and over again.

If it's awesome, submit it to CRAN.

This was an extremely simple version of making a package.

For better details, check out [the free book](http://r-pkgs.had.co.nz/) from Hadley Wikham. 


