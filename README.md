
<!-- README.md is generated from README.Rmd. Please edit that file -->

# rpkgchecker

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental)
<!-- badges: end -->

Provides tools to check R package dependencies, minimum R versions and
comparison with internal copy of package install files.

**rpkgchecker** can be used to generate a list of direct download links
for packages and dependent packages, for example:

  - If need to acquire a new package, this tool will check for any
    dependent packages required.
  - If wish to upgrade to a specific version of a package, this will
    check if available in the CRAN repository to be used for your
    release of R.
  - If wish to acquire a new copy of all packages on a server currently
    and any check for any new dependencies.

## Which copy of CRAN is searched

The process makes use of the base R `available.packages` function.

The `contriburl` argument of `available.packages` allows to specify the
URL for a particular CRAN repository.

In turn, the **rpkgchecker** functions `available_packages_tb` and
`available_packages_long` allow for a specific CRAN repository to be
specified with the `cran_repo_url` argument.

By default `cran_repo_url` is set to search compiled Windows Binary
repositories based on the R release being used while running the
function.

With the default `cran_repo_url` argument, the code below is used to
derive the Windows Binary URL for the current R release and then search
it in `available.packages`:

``` r
search_url <- utils::contrib.url(
      repos = "https://cran.r-project.org/",
      type = "win.binary"
    )
```

For example, if using R 3.6.3, then the `search_url` returned from the
above code will be `https://cran.r-project.org/bin/windows/contrib/3.6`
(assuming not using release of R that is so old that Windows binaries
are no longer stored).

The `available.packages` function then can search this URL returning
package versions and dependencies relevant to that release of R, not the
latest release (see code below). Note `available.packages` is querying a
`PACKAGES` yaml type file that should be present in each repository of R
packages.

``` r
  available_packages <- utils::available.packages(
    contriburl = search_url,
    filters = c("duplicates"), ignore_repo_cache = TRUE
  )    
```

In locked down, Windows-based IT systems, where R release being used may
be older, we think this is the most useful package repository to search.

## Installation

Install the package from GitHub with devtools.

``` r
devtools::install_github("tomwilsonsco/rpkgchecker@main")
```

## Example

This shows a possible workflow using the available functions.

``` r
library(rpkgchecker) # load the rpkgchecker package

# get a tibble of available packages with 1 dependent package per row
available_long_tb <- available_packages_long()

# filter to a specific package, here "fabletools"
# this will get all dependencies by querying its dependency tree recursively
search_tb <- search_requirements(available_long_tb, "fabletools")

# get all existing packages stored on a shared server as windows binaries
server_tb <- existing_server_packages("//s1428a/R_Packages/R_3_6_3_Packages")

# compare the required packages with those on the server currently and indicate
# which need to be acquired or upgraded
compare_tb <- compare_available_server(required_tb, server_tb)

# export a csv file of just those packages need to request
# i.e. not on server currently or need upgrading
request_tb <- export_required_packages(compare_tb, "C:/repos")
```

## Searching dependencies for all existing packages on a server

In locked down IT situations, might have a server inside the
organisation’s firewall storing R packages for install internally.

The functions in this package can be used to refresh the download links
for the latest package versions corresponding to current R release and
check for any new package dependencies.

``` r
# get a tibble of available packages with 1 dependent package per row
available_long_tb <- available_packages_long()

# get all packages stored on a shared server as windows binaries
server_tb <- existing_server_packages("//s1428a/R_Packages/R_3_6_3_Packages")

# search for all existing server packages, depdendent packages 
# and return package versions and links corresponding to current R release.
search_multiple_tb <- search_requirements_multiple(available_long_tb, 
                                                   server_tb$server_package)
```
