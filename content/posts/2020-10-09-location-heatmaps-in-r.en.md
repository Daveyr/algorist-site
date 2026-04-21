---
title: Location heatmaps in R
author: ''
date: '2020-10-09'
# draft: true
slug: location-heatmaps-in-r
categories:
  - Guide
tags:
  - R
  - Mapping
subtitle: ''
summary: ''
authors: []
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

<script src="/rmarkdown-libs/htmlwidgets/htmlwidgets.js"></script>

<script src="/rmarkdown-libs/pymjs/pym.v1.js"></script>

<script src="/rmarkdown-libs/widgetframe-binding/widgetframe.js"></script>

# Location heatmaps in R

Let’s use a built-in example within R: location of earthquakes off the island of Fiji.

    #>      lat   long depth mag stations
    #> 1 -20.42 181.62   562 4.8       41
    #> 2 -20.62 181.03   650 4.2       15
    #> 3 -26.00 184.10    42 5.4       43
    #> 4 -17.97 181.66   626 4.1       19
    #> 5 -20.42 181.96   649 4.0       11
    #> 6 -19.68 184.31   195 4.0       12

This data set contains spatial point data of earthquakes modelled with a certain magnitude and focal depth using observations from a certain number of observatories (`stations`).

# Take a shape and setup the base map

    #> ℹ tmap modes "plot" - "view"
    #> ℹ toggle with `tmap::ttm()`

# Create a grid

This makes an outline and is a good check to see if the bounding box is correct

Now divide into a grid with cells. In the case below we divide the bounding box region
into 20 cells of equal size and equal length in both dimensions.

## Join and aggregate

Form a spatial join with the point observations. Group by `id` of the cell and any group variables in the point observations (such as measurement type, date, object, etc.). Then aggregate, e.g., by summing or calculating the mean of all observations associated with that cell.

## A true heatmap

The above wasn’t really a heatmap so much as a grid. Fortunately the leaflet package has a built in function that is able to render heatmaps properly. Or rather it is contained in the leaflet.extras package.

<div id="htmlwidget-1" style="width:100%;height:480px;" class="widgetframe html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"url":"/posts/2020-10-09-location-heatmaps-in-r.en_files/figure-html//widgets/widget_unnamed-chunk-6.html","options":{"xdomain":"*","allowfullscreen":false,"lazyload":false}},"evals":[],"jsHooks":[]}</script>
