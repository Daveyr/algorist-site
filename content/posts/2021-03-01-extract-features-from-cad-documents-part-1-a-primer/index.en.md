---
title: 'Extract features from CAD documents Part 1: A primer'
author: ''
date: '2021-03-01'
slug: extract-features-from-cad-documents-part-1-a-primer
categories:
  - Blog
  - Guide
tags:
  - CAD
  - Built Environment
  - Python
subtitle: ''
summary: ''
authors: []
lastmod: '2021-03-01T16:50:13Z'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

This is the first part of a series of posts describing my experience of extracting objects from a CAD document.

## Motivation

Recently at work a project came up that involved the optimisation of building occupancy during the refit of a 16 floor central London office block. As we intend to optimise team locations down to the seat level within a floor, we were given floor plans in both pdf and dwg file formats.

The original scope allowed for us to rely on another team to manually encode the distances between each desk on each floor, but there are many downsides to this approach: human error; costly rework when plans change; lack of ability to iteratively improve on feature extraction; and huge risk of schedule overrun when modelling has to wait on this input data. The alternative was to programmatically scrape the CAD file for seat ID and X and Y coordinates. I'll present the solution in [Part 2](https://www.algorist.co.uk/post/extract-features-from-cad-documents-part-2-using-ezdxf/), but first here is what I learned about the structure of CAD files that I needed before writing an extraction utility.

## File types
There are generally two file types in the world of CAD: DWG and DXF. DWG is meant as shorthand for _drawing_, and is a proprietary file format belonging to AutoCAD. It's not very useful on its own so I found it best to convert to DXF using the open source file converter provided by the [Open Design Alliance](https://www.opendesign.com/guestfiles/oda_file_converter).

DXF is short for _design exchange format_, and comes in binary and ASCII flavours. I prefer the ASCII version as it is human readable but the downside is that it is uncompressed and so file sizes have the potential to be large. It is much more easily shared between programs (even Inkscape) and I find that LibreCAD is a really lightweight way of browsing floor plans in this format.

## File structure
A CAD document comprises one model space and zero or more paper spaces. A model space is a limitless field in XYZ space with a certain unit and coordinate frame of reference. A paper space is what you might expect: a layout designed for presentation and printing. Paper spaces are constrained in spatial extent and will have a scale, where the units in model space are scaled for rendering. A perspective is also defined, which affects the perception of distance and distortion (if in 3d).

For extracting objects and their locations we need the model space rather than the paper space.

## Components of a CAD drawing

Building up from the smallest elements, we have...

### Entities

Entities are the most primitive element of a CAD drawing, including points, lines, rectangles, circles and elliptical arcs. 

### Blocks

Blocks are a group of one or more entities. It functions as a template, which can be inserted into the model space multiple times. Each instance of a block is called an insert. If you update a block (by editing an element within the block, for instance) then every insert of that block is updated.

### Layers

A layer organises many elements and inserts under common attributes and under common command (e.g., visibility, locked/unlocked). Whilst a layer may contain many elements, an element can only be on a single layer. This fact makes layers useful for searching for entities and inserts.

## Extraction strategy

For my purpose, I need to extract each seat (or alternatively, each desk) from each floor. This means that I should search for every insert of the appropriate seat block, likely located within a common layer. Ideally that search would return a list containing all the inserts, each of which should contain a unique identifier and an X/Y coordinate.

For that we will use the [ezdxf](https://ezdxf.readthedocs.io/en/stable/) package in Python in [Part 2](https://www.algorist.co.uk/post/extract-features-from-cad-documents-part-2-using-ezdxf/).