
<!-- README.md is generated from README.Rmd. Please edit that file -->

# mpaviewer

<!-- badges: start -->

[![Lifecycle:
experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![R-CMD-check](https://github.com/dbca-wa/mpaviewer/workflows/R-CMD-check/badge.svg)](https://github.com/dbca-wa/mpaviewer/actions)
[![docker](https://github.com/dbca-wa/mpaviewer/actions/workflows/docker.yaml/badge.svg)](https://github.com/dbca-wa/mpaviewer/actions/workflows/docker.yaml)
[![pkgdown](https://github.com/dbca-wa/mpaviewer/actions/workflows/pkgdown.yaml/badge.svg)](https://github.com/dbca-wa/mpaviewer/actions/workflows/pkgdown.yaml)
<!-- badges: end -->

[mpaviewer](https://mpaviewer.dbca.wa.gov.au/) is a dashboard of marine
monitoring data for DBCA staff.

## Architecture

-   Data is loaded from a locally saved `.rds` file, or freshly loaded
    from Google Drive or the DBCA data catalogue (and then saved as
    `.rds`). The locally saved file lives on a persistent volume.

## Data

The dashboard runs off a single `mpa_data.rds` R save file which is
produced from a range of source data files (CSV, TXT, SHP).

### Current data ETL

Archived on the DBCA data catalogue as dataset
[mpaviewer](https://data.dbca.wa.gov.au/dataset/mpaviewer) are: \* A
link to a Google Drive folder which contains a copy of source data files
in the folder/file/data structure expected by
`mpaviewer::generate_data()`. \* Links to the deployed dashboard and
this GitHub code repository \* The mpaviewer data file `mpa_data.rds` at
a specific resource ID expected by `mpaviewer::download_data()`.

#### Update source data

Data owners can update the Google Drive folder with new data.

#### Generate dashboard data

Whenever the source data was updated, an analyst must download the
contents of the Google Drive folder “mpaviewer” into the directory
`inst/data` of a local clone of the mpaviewer codebase and run
`mpaviewer::generate_data()`, then upload the data file `mpa_data.rds`
to the DBCA data catalogue.

#### Update the dashboard with new data

A maintainer accesses the running `mpaviewer` Docker container through a
shell, runs R, and executes
`mpaviewer::download_data(data_dir="/app/inst/data")`.

### Ideal data ETL

A second Docker container should run
`mpaviewer::download_data(data_dir="/app/inst/data")` as a cronjob,
following
[this](https://github.com/dbca-wa/etlTurtleNesting/tree/master/cron)
example.

The container could also run other R scripts, e.g. scripts downloading
source data from the DBCA data catalogue, then running
`mpaviewer::generate_data()` on the downloaded copies. The downloading,
processing, and uploading of data can be split out into another R
package or remain in mpaviewer.

Data owners could further automate their data pipelines by scripting the
upload of their source data to the data catalogue, e.g. using R.

## Development

`mpaviewer` is an RShiny app using [Shiny
modules](https://shiny.rstudio.com/articles/modules.html), developed
using [`golem`](https://mastering-shiny.org/scaling-modules.html).

The commands to develop new functionality are in `dev/02_dev.R`.

## Release

Once app and Docker image work, create a new version, tag, and push the
tag.

``` r
styler::style_pkg()
spelling::spell_check_package()
spelling::update_wordlist()

# Code and docs tested, working, committed
usethis::use_version(which="patch")
usethis::use_version(which="minor")
usethis::use_version(which="major")
usethis::edit_file("NEWS.md")

# Document to load new package version. Git commit, tag, and push.
devtools::document()
v <- packageVersion("mpaviewer")
system(glue::glue("git tag -a v{v} -m 'v{v}'"))
system(glue::glue("git push && git push --tags"))
```

## Deployment

GitHub Actions are configured to build a new Docker image for every tag
starting with “v”, such as “v1.0.0”. The images are pushed to the GitHub
container registry `ghcr.io`. The DBCA maintainer of `mpaviewer` then
updates the tag number (e.g. `1.0.0`) in the image run by Kubernetes,
which will trigger a hot reload of the image.

The commands to deploy a new version are in `dev/03_deploy.R`.
