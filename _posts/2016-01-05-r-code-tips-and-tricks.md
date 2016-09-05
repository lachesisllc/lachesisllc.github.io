---
layout: inner
author: Chris
title: 'R Code Tips and Tricks'
categories: ["big data", "development"]
tags: ["big data", "development"]
featured_image: '/images/r_code_tips.png'
---

We’ve been building quite a number of exciting Machine Learning projects hosted in Azure over the past year.  Throughout the process we’ve found a number of Tips and Tricks that will help make a smooth transition from development to production.  I’ll keep this post updated with the non-obvious ones as well as link to great resources for quick reference.

### QUICK REFERENCES

[Google’s R Style Guide](https://google.github.io/styleguide/Rguide.xml) <br/>
[Author custom R modules in Azure Machine Learning](https://azure.microsoft.com/en-us/documentation/articles/machine-learning-custom-r-modules/)<br/>
[R Studio](https://www.rstudio.com/)

#### USE THE GOOGLE CODING NAMING STANDARDS – BUT WITH THE FOLLOWING CAVEATS!


* Do NOT create variable names containing periods (i.e. some.string <- “hello”)<br/>
Periods work well in the custom execute R script module and will also work in a script bundle, but it will fail miserably if you try to convert your code to a custom R module.The Azure ML custom R module compiler will throw a non-descriptive error code 0114 compiler error if any variable contains a dot.  Instead use the alternative pascal casing in the standards (i.e. someString <- “hello”)
* Do NOT create globally scoped variables<br/>
Setting up global variables in one source file, then trying to access them in another that references it works in the execute R script module and in script bundles – but will fail to compile if you convert your code to a custom R module.For example, consider the following two scripts:<br/>

~~~ R
 common-functions.R
 …
 commonDateTimeFormat <- “%m/%d/%Y %H:%M”
 …
 convert-dateTime.R
 …
 source(“src/common-functions.R”)
 result <- as.POSIXlt(stringToConvert, commonDateTimeFormat, tz=””))
 …
~~~

The Azure ML custom R module compiler will throw a non-descriptive error code 0114 compiler error if the variable is assigned in a different script file and then referenced in the current script.
* Do NOT inline function calls into return statements<br/>
We’ve seen issues with the Custom R module Azure ML compiler when in-lining predict calls for models in return statements (i.e. return(predict(linearModel, …)).  This has consistently returned the dreaded 0114 error code, while other calls such as return(as.POSIXlt(…)) seem to work.  Do to the inconsistency, we suggest assigning the result and return the simple variable it is assigned to.

#### R STUDIO PROJECT STRUCTURE

First – if you aren’t familiar with R Studio – grab a copy, I promise you’ll fall in love.

* Do create a consistent folder structure that accounts for the “src/foo.R” structure Azure ML uses<br/>
We’ve found the following structure lends itself well to creating script bundles and custom R modules in Azure ML
Root
  * analysis.R [Main entry point for local experiments]
  * environment-bootstrap.R [Installs required packages if missing and sets up the testthat framework]
  * data/ – [Data for experiments]
  * doc/ – [Source files for creating word / pdf / html documents via Rmd]
  * figs/ – [R files for creating plots used in analysis.R and doc generation]
  * output/ – [Landing place for generated artifacts – we exclude this directory from source control]
  * src/ – [R functions and Custom R Module xml files]
  * tests/ – [Unit tests for files in the src directory]


#### SUMMARY

I’ll keep this up to date with the non-obvious tips and tricks as the team comes across as we continue to learn more.  I hope this helps you create exciting and invaluable new Azure ML solutions!