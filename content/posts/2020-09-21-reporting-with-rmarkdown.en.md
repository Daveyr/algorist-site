---
title: Reporting with Rmarkdown
author: ''
date: '2020-09-21'
slug: reporting-with-rmarkdown
categories:
  - Guide
tags:
  - R
subtitle: ''
summary: ''
authors: []
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
output: html_document
---

Rmarkdown was a revelation to me when I was first introduced to it in SEAMS (now Arcadis Gen). I'd used Jupyter notebooks before for Python and loved the live lab notebook feel of them. Rmarkdown for R is like this but more polished, more final, more suited to a corporate or public end user. It also has a few tricks up its sleeve.

## Reproducibility
Much like Jupyter notebooks, the biggest draw for RMarkdown is the reproducibility. Any other collaborator can take your document and re-render it using the same input data to produce the same output. The corollary is that if input data or assumptions change then it is very straightforward to rerun with no extra effort. For instance, sharing reports within a team may require your code chunks to be visible (just like Jupyter) but for external reporting these may need to be set to invisible (using one line of code):


``` r
knitr::opts_chunk$set(echo = FALSE, eval = FALSE)
```

## Flow
For a similar reason to the above, having code so close to commentary helps with flow. I've taken to doing exploratory data analysis within an Rmarkdown document that will eventually become my interim report document. This method requires some discipline up front (read: constant revision, simplification and lots of code comments!) but it can be a really helpful way of generating an artefact from what is a trial-and-error exploratory process.  

## Flexdashboards
I've had limited exposure to using `flexdashboard`, but the principle is to render a mostly static dashboard with a few limited widgets and interactivity (such as leaflet maps). This is achieved with some minor additional markdown syntax and a changed YAML header like the one below.

<pre>
---
title: "Untitled"
output: 
  flexdashboard::flex_dashboard:
    orientation: columns
    vertical_layout: fill
---
</pre>

The real benefit of flexdashboard over more interactive dashboards is that it is a portable document that doesn't require a client-server architecture.

## Interactive dashboards with Shiny
When you really need filters, sliders, realtime data feeds and all sorts of interactivity and you are prepared to host it on a remote server then Shiny is the way to go. The code is essentially split into how the app looks (referred to as the `ui` object) and how the app processes (referred to as the `server` object). This can either be written in the same file, with the structure below, or split into `ui.R` and `server.R` files.



You don't need Rmarkdown to develop Shiny apps but if you want the same widgets in Rmarkdown you need to add `runtime: shiny` to the YAML header.

## Batch reporting using parameters
The culmination of reproducibility and analytical flow is the concept of _write once render many_. Similar to DRY (don't repeat yourself), nothing beats preparing a single template document that can be automatically tweaked to generate lots of similar - but crucially unique - reports. This could be split by region, business unit, financial quarter, you name it.  Fortunately Rmarkdown is set up to easily do this.

Within the YAML header at the top of the Rmarkdown document you can specify a `params:` tag with _name: value_ pairs underneath. Within the code in the document, a list object called `params` is available containing the name and values specified in the header. Below is an example of an Rmarkdown document set up to report on the `mtcars` dataset built in R, filtered according to the cylinder value stored in `params$cyl`.

### Rmarkdown document

<pre>
````
---
title: "mtcars"
author: "Richard Davey"
date: "22/09/2020"
params:
  cyl: 4
output: html_document
---

## Mtcars

With parameterised reporting it is possible to feed in parameters using the `params:` section in the YAML header, and use these values within a `params` list object. This example uses the number of cylinders to filter the `mtcars` dataset and plot and show this filtered dataset. Common use cases within business could be reporting by region or by financial quarter.

```{r cars, message=FALSE, warning=FALSE}
library(dplyr)
library(knitr)
library(ggplot2)

df <- mtcars %>%
  filter(cyl == params$cyl)

kable(df, caption = paste0("Table of cars with cyl == ", params$cyl))

```

## Including Plots

You can also embed plots, for example:

```{r plot, echo=TRUE, message=FALSE, warning=FALSE}
ggplot(df, aes(x = disp, y = mpg)) +
  geom_point() +
  geom_smooth(method = "lm") +
  labs(title = paste0("Relationship between mpg and disp for ", params$cyl, " cylinder cars"))
```

````
</pre>

### Batch reporting script

Once the document is created the code below will batch generate reports for all valid cylinder values.




Finally, if you needed even more endorsement you can actually write websites using Rmarkdown, just like this one. See [blogdown](https://bookdown.org/yihui/blogdown/) for details.
