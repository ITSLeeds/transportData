transportdata
================

-   [1 Installation](#installation)
-   [2 Datasets](#datasets)
    -   [2.1 Origin-destination data](#origin-destination-data)
    -   [2.2 Centroid data](#centroid-data)
    -   [2.3 Route data](#route-data)
-   [3 Code of Conduct](#code-of-conduct)

<!-- README.md is generated from README.Rmd. Please edit that file -->
<!-- badges: start -->
<!-- [![Lifecycle: experimental](https://img.shields.io/badge/lifecycle-experimental-orange.svg)](https://www.tidyverse.org/lifecycle/#experimental) -->

[![CRAN
status](https://www.r-pkg.org/badges/version/transportdata)](https://CRAN.R-project.org/package=transportdata)
[![R-CMD-check](https://github.com/ITSLeeds/transportdata/workflows/R-CMD-check/badge.svg)](https://github.com/ITSLeeds/transportdata/actions)
<!-- [![Codecov test coverage](https://codecov.io/gh/ITSLeeds/transportdata/branch/master/graph/badge.svg)](https://codecov.io/gh/ITSLeeds/transportdata?branch=master) -->
<!-- badges: end -->

`transportdata` is a package containing transport datasets for teaching,
methodological research and software development (e.g. for
benchmarking). The aim is for it to be for people working with transport
data what `spData`, a package used for teaching and demonstrating
spatial data methods, is for people working with spatial data. The
package will be especially useful for transport/mobility researchers
using R.

# 1 Installation

<!-- You can install the released version of transportdata from [CRAN](https://CRAN.R-project.org) with: -->
<!-- ``` r -->
<!-- install.packages("transportdata") -->
<!-- ``` -->

Install the development version from [GitHub](https://github.com/) with:

``` r
# install.packages("devtools")
devtools::install_github("ITSLeeds/transportdata")
```

# 2 Datasets

The datasets included in the package, and code chunks used to create
them, are outlined below. After you have installed the package, the
datasets should be available.

``` r
library(transportdata)
```

To reproduce the following code, you will need to to have installed a
number of packages. The following packages must also be loaded:

``` r
library(dplyr)
library(stplanr)
```

## 2.1 Origin-destination data

OD data is arguably the most fundamental type of data for representing
city to global transport systems. The OD dataset used for this package
represents the number of commutes between a subset of desire lines
between administrative zones in Leeds, UK. The dataset was generated as
follows:

``` r
od_data = pct::get_od()
# [1] 2402201
nrow(od_data)
od_data_leeds = od_data %>% 
  filter(la_1 == "Leeds") %>% 
  filter(la_2 == "Leeds") 
nrow(od_data_leeds)
# [1] 35463
od_leeds = od_data_leeds %>% 
  filter(all >= 50)
usethis::use_data(od_leeds, overwrite = TRUE)
```

``` r
dim(transportdata::od_leeds)
#> [1] 948  18
```

## 2.2 Centroid data

``` r
centroids = pct::get_centroids_ew()
centroids_leeds = centroids %>% 
  filter(stringr::str_detect(string = msoa11nm, pattern = "Leeds")) %>% 
  select(geo_code = msoa11cd) %>% 
  sf::st_transform(4326)
usethis::use_data(centroids_leeds, overwrite = TRUE)
```

``` r
dim(centroids_leeds)
#> [1] 107   2
```

## 2.3 Route data

The route data was generated as follows:

``` r
od_interzonal = od_leeds %>% 
  filter(geo_code1 != geo_code2)
desire_lines_leeds = od::od_to_sf(od_interzonal, centroids_leeds)
#> 0 origins with no match in zone ids
#> 0 destinations with no match in zone ids
#>  points not in od data removed.

summary(sf::st_length(desire_lines_leeds))
#> Linking to GEOS 3.8.0, GDAL 3.0.4, PROJ 7.0.0
#>    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#>     595    1878    3493    4219    5809   18409
```

``` r
routes_leeds_foot = route(l = desire_lines_leeds, route_fun = route_osrm, osrm.profile = "foot")
saveRDS(routes_leeds_foot, "routes_leeds_foot.Rds")
piggyback::pb_upload("routes_leeds_foot.Rds")

routes_leeds_bike = route(l = desire_lines_leeds, route_fun = route_osrm, osrm.profile = "bike")
saveRDS(routes_leeds_bike, "routes_leeds_bike.Rds")
piggyback::pb_upload("routes_leeds_bike.Rds")

routes_leeds_car = route(l = desire_lines_leeds, route_fun = route_osrm, osrm.profile = "car")
saveRDS(routes_leeds_car, "routes_leeds_car.Rds")
piggyback::pb_upload("routes_leeds_car.Rds")
```

``` r
routes_leeds_foot = get_td("routes_leeds_foot")
#> Reading in the file from https://github.com/ITSLeeds/transportdata/releases/download/0.1/routes_leeds_foot.Rds
routes_leeds_bike = get_td("routes_leeds_bike")
#> Reading in the file from https://github.com/ITSLeeds/transportdata/releases/download/0.1/routes_leeds_bike.Rds
routes_leeds_car = get_td("routes_leeds_car")
#> Reading in the file from https://github.com/ITSLeeds/transportdata/releases/download/0.1/routes_leeds_car.Rds
```

``` r
rnet_leeds_foot = overline(routes_leeds_foot, "foot")
plot(rnet_leeds_foot["foot"], lwd = rnet_leeds_foot$foot / 500)
rnet_leeds_bike = overline(routes_leeds_bike, "bicycle")
plot(rnet_leeds_bike["bicycle"], lwd = rnet_leeds_bike$bicycle / 500)
rnet_leeds_car = overline(routes_leeds_car, "car_driver")
plot(rnet_leeds_car["car_driver"], lwd = rnet_leeds_car$car_driver / 500)
```

<img src="man/figures/README-routes-rnet-1.png" width="33%" /><img src="man/figures/README-routes-rnet-2.png" width="33%" /><img src="man/figures/README-routes-rnet-3.png" width="33%" />

# 3 Code of Conduct

Please note that the transportdata project is released with a
[Contributor Code of
Conduct](https://contributor-covenant.org/version/2/0/CODE_OF_CONDUCT.html).
By contributing to this project, you agree to abide by its terms.