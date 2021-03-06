
<!-- README.md is generated from README.Rmd. Please edit that file -->
    ## Loading stemhelper

stemhelper: Helper functions for STEM loading, mapping, plotting, and analysis
==============================================================================

<!-- [![License: GPL v3](https://img.shields.io/badge/License-GPL%20v3-blue.svg)](http://www.gnu.org/licenses/gpl-3.0) -->
**THIS PACKAGE IS UNDER ACTIVE DEVELOPMENT. FUNCTIONALITY MAY CHANGE AT ANY TIME.**

The goal of stemhelper is to provide functionality for loading, mapping, plotting, and analyzing STEM results.

Installation
------------

You can install stemhelper from GitHub with:

``` r
# install the development version from GitHub
# install.packages("devtools")
devtools::install_github("CornellLabofOrnithology/stemhelper")
```

Vignette
--------

For a full introduction and advanced usage, please see the package [website](https://cornelllabofornithology.github.io/stemhelper). An [introductory vignette](https://cornelllabofornithology.github.io/stemhelper/articles/stem-intro-mapping.html) is available, detailing the structure of the results and how to begin loading and mapping the results. Further, an [advanced vignette](ttps://cornelllabofornithology.github.io/stemhelper/articles/stem-pipd.html) details how to access additional information from the model results about predictor importance and directionality, as well as predictive performance metrics.

Quick Start
-----------

After downloading results...

One of the first things to do is to load a RasterStack of estimated abundance predictions and plot a week of estimates.

``` r
library(viridis)
library(raster)

# TODO set this up to do a curl of the example results first

# SETUP PATHS
# Note, if you currently access results you can use this path construction
# by replacing the root_path with where your copy lives
root_path <- "~/Box Sync/Projects/2015_stem_hwf/documentation/data-raw/"
species <- "woothr-ERD2016-PROD-20170505-3f880822"
sp_path <- paste(root_path, species, sep = "")

# load a stack of rasters with the helper function stack_stem()
abund <- stack_stem(sp_path, variable = "abundance_umean")

# subset to the 26th week of the year
abund26 <- abund[[26]]

# project to Mollweide for mapping and extent calc
mollweide <- CRS("+proj=moll +lon_0=-90 +x_0=0 +y_0=0 +ellps=WGS84")
abund_moll <- projectRaster(abund26, crs = mollweide)

# calculate the full annual extent
# and set dimensions to set plotting size
sp_ext <- calc_full_extent(abund_moll)

xlimits <- c(sp_ext[1], sp_ext[2])
ylimits <- c(sp_ext[3], sp_ext[4])

xrange <- xlimits[2] - xlimits[1]
yrange <- ylimits[2] - ylimits[1]

# these are RMarkdown specific here, but could be passed to png()
w_cm <- 8.5
h_cm <- w_cm*(yrange/xrange)
knitr::opts_chunk$set(fig.width = w_cm, fig.height = h_cm)

# calculate ideal color bins for abundance values across the full year
year_bins <- calc_bins(abund_moll)

# to add context, pull in some reference data to add
wh <- rnaturalearth::ne_countries(continent = c("North America",
                                                "South America"))
wh_states <- rnaturalearth::ne_states(iso_a2 = unique(wh@data$iso_a2))
wh_moll <- sp::spTransform(wh, mollweide)
wh_states_moll <- sp::spTransform(wh_states, mollweide)

# start plotting
par(mfrow = c(1, 1), mar = c(0, 0, 0, 6))

# use the extent object to set the spatial extent for the plot
sp::plot(sp_ext, col = 'white')

# add background spatial context
sp::plot(wh_states_moll, col = "grey70", add = TRUE)

# plot the raster
plot(abund_moll,
     maxpixels = ncell(abund_moll),
     ext = sp_ext,
     breaks = year_bins,
     col = viridis::viridis(length(year_bins-1)),
     xaxt = 'n',
     yaxt = 'n',
     add = TRUE,
     legend = TRUE,
     legend.shrink = 0.97,
     legend.width = 2,
     axis.args = list(at = year_bins,
                      labels = round(year_bins, 2),
                      cex.axis = 0.7))

# add state boundaries on top
sp::plot(wh_states_moll, add = TRUE, border = 'black', lwd = 1)
```

<img src="README-quick_start-1.png" style="display: block; margin: auto;" />
