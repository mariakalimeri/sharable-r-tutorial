
<!-- README.md is generated from README.Rmd. Please edit that file -->
Writing sharable (R) code. Uh... and sharing it!
================================================

In this short tutorial we will build an R package and a drat repository to host it, using GitHub as a Web Server. At the end we will set up Travis CI to automatically push updates of our package into the drat repository.

Prerequisites:
--------------

-   (Optional) Rstudio; this is not necessary to produce R packages but it does make your life easier as the main package for R package development, `devtools`, is well integrated with RStudio.
-   The following R packages:
    -   `devtools` and `roxygen2` in order to build R packages
    -   `tidyverse`, `rlang`
    -   `drat`
-   (Optional) A github account. If you wish to replicate steps X-X herein you need your own github account.

Create an R package
-------------------

I usually like to jump right into the essence of it, so let's start by building a minimal R package following [Hilary Parker's building steps](https://hilaryparker.com/2014/04/29/writing-an-r-package-from-scratch/). Afterwards, we can discuss when or why build an R package along with some good practices. Most of the discussed topics below are taken from Hadley Wickham's [R packages](http://r-pkgs.had.co.nz/).

### Step 1: Create a package directory

Hilary is building a cat-themed package, but in order to not discourage dog persons, I will go with a different theme. Let's make a package whose aim is to perform some basic statistics on blood metabolomics data. (In fact, other than the slightly grotesque namings, the package's single demo utility has nothing specific to blood metabolomics; we will however add simulated metabolomics dataset in our package's examples later on.)

Let's create the minimum amount of subdirectories your package needs.

``` r
# Navigate to the desired parent directory
setwd("parent_directory")
# Create the directory of your package with the minimum amount of subdirectories
# Its reasonable but not necessary to name the directory with the same name as 
# the package 
usethis::create_package("bloodstats")
```

Alternatively, from within `RStudio`, you may perform the step above by going to `File` -&gt; `New Project...` -&gt; `New Directory` -&gt; `R package`, type in the package name and location and click `Create Project`. Notice, you may also add existing functions at this step.

What the above function did was to create a directory called `bloodstats/`, inside the `parent_directory/`, and inside `bloodstats/` two subdirectories, - `R/`, that will contain the package source code and - `man/`, that will contain the package's documentation. Finally, there are two files, `DESCRIPTION` and `NAMESPACE`. Go ahead and edit the `DESCRIPTION` file with a short description of the package, your name and contact information, etc. This is the file where you'll also be keeping track of your package versioning. [The package namespace](http://r-pkgs.had.co.nz/namespace.html), as recorded in the `NAMESPACE` file, is something you should understand if you plan to share your packages. Namespace takes care of imports and exports such that your package will coexist in harmony with other packages. The file `NAMESPACE` is something you shouldn't edit by hand, instead `roxygen2` will take care of updating this file everytime you build your documentation.

An important note concerning the name of your package, especially if you plan to share it with others. It's good to make sure that the name of your package is not already in use by another CRAN package. You can check this by loading `https://cran.r-project.org/web/packages/bloodstats`.

### Step 2: Add functions

Below is an example of a function that fit's the scope of our package. It takes a data frame as input (supposedly containining blood biomarkers) and returns the mean value of each column (variable) as long as this is numeric.

``` r
bloodmeans <- function(df) {
  df %>%
    dplyr::summarise_if(is.numeric, mean, na.rm = TRUE)
}
```

Save this function as `bloodmeans.R` inside the `R` subdirectory.

### Step 3: Add documentation

What you need to do is type each function's description and other comments at the beginning of each function in the form of special comments and `roxygen2` will take care of building the whole documentation. An example of how the special comments that constitute object documentation should be is shown below. For more information on the subject see [here](http://r-pkgs.had.co.nz/man.html).

``` r
#' Extract Mean Values of Blood Biomarkers  
#' 
#' This function accepts a dataframe as input and extracts the mean value of 
#' each numeric variable.
#' 
#' @param df a \code{data.frame} with at least one numeric variable in order to 
#' get a non-empty result.
#' @return a data.frame with the mean values of each numeric 
#' @importFrom dplyr summarise_if
#' @author John Doe 
#' @export 
#' @examples 
#' data.frame(er = c(1,2,3), c(4,5,6)) %>% 
#'   bloodstats::bloodmeans()
bloodmeans <- function(df) {
  df %>%
    dplyr::summarise_if(is.numeric, mean, na.rm = TRUE)
}
```

It is not necessary to have each function in its own file - although it usually makes the code easier to read/access by others - but when you add more than one function in a file make sure to add the documentation for each function just before its definition.

Some notes on documentation. I find it always very usefull to have a working example. This often may need one or two built-in datasets. These help to demonstrate the input that the function expects and to have a working example that is short (i.e. you don't have to generate data in the example to run the function).

### Step 4: Process documentation

You can now use `devtools::document()` to build your documentation. From within the package directory, type the following:

``` r
# If you are using an RStudio project for your package development you are most 
# likely already in the package directory. If not, navigate into it
# > setwd("./bloodstats")
# and type:
devtools::document()
```

This function is a wrapper for `roxygen2::roxygenize()`; it adds `.Rd` to the `man` directory, one for each object in your package, assuming you have written comments as suggested in step 3. The function will also update the `NAMESPACE` file of the main directory with the corresponding imports and exports.

Remember to update your package's version from 0.0.0.9999 to 0.0.1, if you feel it's time.

### Step 5: Write tests

This is an important part of package development. The main aims of writing formal tests is to make sure, that you will not break code that used to work, when you come back in the future to add features or improve existing code.

You can use `usethis::use_testthat()` to set up the package to use tests. This command will do all the necessary steps below:

1.  Create a tests/testthat directory.
2.  Adds testthat to the Suggests field in the DESCRIPTION.
3.  Create a file tests/testthat.R that runs all your tests when R CMD check runs. (See more on automated checking [here](http://r-pkgs.had.co.nz/check.html#check).)

The next step is to actually write the tests. We have only one function at the moment, `bloodmeans()`. Create an R file with the name `test-bloodmeans.R` and type the following contents.

``` r
context("bloodmeans")

library(magrittr)

res <-
  data.frame(var1 = c(1, 2, 3), var2 = c(4, 5, 6)) %>%
  bloodstats::bloodmeans()

test_that("bloodmeans returns output of expected class", {

  expect_true(
    class(res) == "data.frame"
  )
})

test_that("bloodmeans returns expected result given input", {

  expect_true(
    all(res == data.frame(var1 = 2, var2 = 5))
  )
})
```

You may now run the tests as shown below:

``` r
devtools::test()
```

Or you may use the RStudio build-in shortcuts. Refer to the related section in [R-packages:tests](http://r-pkgs.had.co.nz/tests.html) for more info on proper unit testing.

### Step 6: Run checks

After your tests have passed, you should perform another important step of package development, running checks. `devtools::check()` or [`R CMD check`](http://r-pkgs.had.co.nz/check.html#check) will check your code for common issues like documentation mismatches, missing imports etc, including pass or fail of unit tests if such exist.

You should probably run checks quite a bit more often than tests, as this will help to start curing problems and incosistencies as soon as they appear rather than having to deal with a huge amount of them at a much later stage.

So run the command below

``` r
devtools::check()
```

or use the RStudio build-in shortcuts if you prefer.

### Step 7: Install your package

From the parent directory, that contains the `bloodstats` folder, type the following.

``` r
setwd("../")
devtools::install("bloodstats")
```

That will get your package installed in your machine. You can try viewing the documentation of your function by typing

``` r
?bloodmeans
```

Finally, let us now built the source package, we will need this file later in order to insert the package into `drat`.

``` r
# Assuming your current working directory is the bloodstats
devtools::build()
```

This will create the file `bloodstats_0.0.1.tar.gz` in the parent directory of `bloodstats`.

Share your R package
--------------------

### Step 8: Make the package a GitHub repo

If you want to reproduce the steps from here onwards you will need a [github](https://github.com/) account.

Just as Hilary in her post, we will not dive into git and GitHub here (let me also refer to [Karl Broman’s Git/GitHub Guide](http://kbroman.org/github_tutorial/) for that). For the purposes of this tutorial, I will assume some basic knowledge of git. If you don't have it, it's ok, you may simply copy-paste the git commands here, and come back to it later for details.

#### Step 8a: Push initial commit

Let us follow the steps [in this guide](https://help.github.com/en/articles/adding-an-existing-project-to-github-using-the-command-line) in order to create a GitHub repository for our existing R package. Do make the following addition though: between steps 4 and 5, add a file with the name `.gitignore` with at least the following contents:

``` r
# example of .gitignore contents for an R package repository
.RData
.Rhistory
.Rproj.user
.Ruserdata
# As an macOS user I also gitignore .DS_Store
```

`.gitignore` contains the names of files that you don't want to include in your git repository; so instead of not staging them every time you commit, you permanently ignore them by adding them in this file.

#### Step 8b: Add a README

This is an important step so that people that land in your github repository page will have an overview of your project.

As R users, it's handy to use an `Rmarkdown` document to write our description. Create a file called `README.Rmd` (add this into a file called `.Rbuildignore` in the top level of the package directory so that `devtools::check()` wont give you additional notes about non-standard files in your package folder) with the contents suggested below;

    ---
    output: github_document
    ---

    <!-- README.md is generated from README.Rmd. Please edit that file -->

    A package with utilities for basic statistics on blood biomarkers.

It's good to enrich your `README` with quick, getting-started examples and other notes. I often try to browse around other GitHub repositories to explore good practices, see for example the `README` file for [`patchwork`](https://github.com/thomasp85/patchwork).

Add, commit and push your newly created `README.Rmd` and `README.md` files.

#### Step 8c: Enable Travis-CI for the packages repository

This step requires that

### Step 9: Create a `drat` repository in Github and push it in github

[source: Drat for package authors](http://eddelbuettel.github.io/drat/DratForPackageAuthors.html)

As a package author with a given GitHub account, in order to create your own R package drat repository, all that is needed is a GitHub repository named `drat` and inside it a the subdirectory `src/contrib/`.

In a terminal, let's create the `drat` folder with its contents and initialize a git repository for it.

    mkdir drat 
    cd drat
    mkdir src
    cd src
    mkdir contrib

Follow the same steps as in *Step 6a* above, in order to create a git repository for this drat and push it in gihub.

The next step is to place packages into drat.

``` r
## insert the bloodstats bundle into the drat repo on local file system
drat::insertPackage("myPkg_0.5.tar.gz", "/srv/projects/git/drat")
```

### Step 10: Add Travis CI support to your drat repository

[source: r-travis](https://github.com/craigcitro/r-travis)

### Step 11: Combining Drat and Travis CI

[source: Use Travis CI for automatic pushes of succesfull builds into drat](https://cran.r-project.org/web/packages/drat/vignettes/CombiningDratAndTravis.html)

Improve package and update!
---------------------------

Let us add an extra input parameter, `var`, in `bloodmeans()`. If `var` is specified, and it's a valid `df` variable, the function will attempt to group the input data frame by it and return means for each group.

Update `bloodmeans.R` with the following, bump your packages version and git add, commit and push your updates.

``` r
#' Extract Mean Values of Blood Biomarkers  
#' 
#' This function accepts a dataframe as input and extracts the mean value of 
#' each numeric variable.  
#' 
#' @param df a \code{data.frame} with at least one numeric variable in order to 
#' get a non-empty result.
#' @param var NULL (default) or the unquoted name of the variable by which to 
#' group the input df.
#' @return a data.frame with the mean values of each numeric.
#' @importFrom dplyr summarise_if
#' @author John Doe 
#' @export 
#' @examples 
#' data.frame(er = c(1,2,3), c(4,5,6)) %>% 
#'   bloodstats::bloodmeans()
bloodmeans <- function(df, var = NULL) {

  # Quote input
  var <- rlang::enquo(var)

  if (!rlang::quo_is_null(var)) {
    df <-
      df %>%
      dplyr::group_by(!!var)
  }

  df %>%
    summarise_if(is.numeric, mean, na.rm = TRUE)
}
```

Further discussion on package development
-----------------------------------------

-   When to build a package. Practical use cases:
    -   Data reading and cleaning functions
    -   Data analysis workflow with a demo vignettes
-   Writing tests
-   Package scope. Should a package have an overall scope? Is a package with a bunch of relatively unrelated functions or a not-well-defined scope better than no package at all?
-   Documentation and package datasets. Importance of examples and links to functions from other packages.
-   Styling your code, e.g. `styler 1.0.0`

References
----------

-   [R packages by Hadley Wickham](http://r-pkgs.had.co.nz/)
-   [Karl Broman’s Git/GitHub Guide](http://kbroman.org/github_tutorial/)
-   [How to create a GitHub repo from an existing directory](https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/)
-   [How to create a GitHub page for your project](https://help.github.com/categories/github-pages-basics/)
-   [Tidy evaluation](https://cran.r-project.org/web/packages/dplyr/vignettes/programming.html) and [most common actions](https://edwinth.github.io/blog/dplyr-recipes/)
-   Make an elegant and useful website for your package with [`pkgdown`](https://pkgdown.r-lib.org/articles/pkgdown.html)
-   [R code styler](https://www.tidyverse.org/articles/2017/12/styler-1.0.0/)
-   [Software semantic versioning](https://semver.org/)
