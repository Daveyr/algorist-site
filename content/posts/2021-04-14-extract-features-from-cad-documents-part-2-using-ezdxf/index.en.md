---
title: 'Extract features from CAD documents Part 2: Using ezdxf'
author: ''
date: '2021-04-14'
slug: extract-features-from-cad-documents-part-2-using-ezdxf
categories:
  - Guide
  - Blog
tags:
  - Built Environment
  - Python
  - CAD
subtitle: ''
summary: ''
authors: []
lastmod: '2021-04-14T16:10:14+01:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In this second part of the series of posts on extracting objects from a CAD document, I'll go through the process of using the `ezdxf` package to implement the extraction strategy discussed in [part one](https://www.algorist.co.uk/post/extract-features-from-cad-documents-part-1-a-primer/).

## Prologue
In part one we looked at the structure of a CAD file and built up a strategy to extract seat types and locations from an architect's floor plan. The motivation for this is to provide seat location data to a model that creates a stack plan with optimal locations of teams to office amenities and other teams they collaborate with.

In summary, our strategy is:

* load the DXF file;
* create an object from the model space;
* build a layer and block type query;
* extract unique ID, x and y features from every element returned from the query, recording the block type and layer it came from;
* check for errors and write to csv file.

## ezdxf
Since the genetic algorithm we've built is in R, I tried to build an extraction tool in R. However, the only DXF file loader package I could find is called [ezdxf](https://ezdxf.readthedocs.io/en/stable/) and is built in Python. I'm not a fan of reinventing the wheel so I wrote a custom package in Python that is built on top of `ezdxf`'s classes and methods.

The following functions are building blocks to load the model space from a dxf file, query the model space object for block inserts that belong to a certain block type or layer, and to extract them. They rely heavily on the documentation and tutorials from the `ezdxf` readme website so I would encourage the interested reader to refer to those documents for background information.

### Load file

```python
import ezdxf
import numpy as np
import pandas as pd
import sys

def load(filepath):
    """
    Loads the modelspace of a dxf file using the ezdxf package.

    Parameters
    ----------
    filepath: string
        The file path to a dxf file

    Returns
    An exdxf modelspace object.
    -------

    """

    try:
        doc = ezdxf.readfile(filepath)
        msp = doc.modelspace()
    except IOError:
        print(f'Not a DXF file or a generic I/O error.')
        sys.exit(1)
    except ezdxf.DXFStructureError:
        print(f'Invalid or corrupted DXF file.')
        sys.exit(2)
    return msp
```

### Print blocks that belong to a layer

```python
def print_blocks(model, layer):
    """
    Prints all blocks belonging to a layer

    Parameters
    ----------
    model : ezdxf modelspace object
        A modelspace object created from a loaded dxf file. 
        See the ezdxf help file for 'readfile' and 'modelspace' methods
        
    layer : string
        A string specifying the layer name in the modelspace object
    
    Returns
    A list of string elements
    -------

    """   
    query_string = '*[layer ? \"%s\"]' % layer
    entities = model.query(query_string)
    layers = set(i.dxf.layer for i in entities)
    for i in layers:
        layer_entity = model.query('*[layer == \"%s\"]' % i)
        layer_entity = [x for x in layer_entity if x.has_dxf_attrib('name')]
        blocks = set(j.dxf.name for j in layer_entity)
        print(i + ":")
        for x in blocks:
            print(x)
```
### Extract inserts belonging to a block type or layer

The first function extracts all inserts of all blocks based on a query made on the model object. Within the body of the function there is code to check for negative x values and mirror them if it finds any. This is because when I compared the DWG and DXF files I noticed that some objects had been mirrored in the DXF file for some reason. This fix worked for me but it may not work for you, depending on your version of CAD software you used, the conversion software and the point of origin (the 0,0 point). This issue is not due to Python as the issue appeared in the input DXF file prior to extraction. You have been warned!

```python
def extract_query(model, query_string, scaling = 0.001):
    """
    Extracts all objects from an ezdxf model object that are returned by a query.

    Parameters
    ----------
    model : ezdxf modelspace object
        A modelspace object created from a loaded dxf file. 
        See the ezdxf help file for 'readfile' and 'modelspace' methods
        
    query_string : string
        A string specifying a layer or block based query
        
    scaling : positive real value
        Depending on the units of the modelspace and the desired units, a scaling
        may be preferred. The default is a unit of mm and a desired unit of metre.
         (Default value = 0.001)

    Returns
    Pandas dataframe object with ID, x and y coordinate values. One row per object.
    -------

    """
    entities = [x for x in model.query(query_string) if x.has_dxf_attrib('name')]
    coords = [i.dxf.insert for i in entities]
    id = ['UID' + i.dxf.handle for i in entities]
    output = pd.DataFrame(coords, columns = ['x', 'y', 'z'], index = id).drop(['z'], axis=1)
    output.index.name = "id"
    output['type'] = [i.dxf.name for i in entities]
    output['layer'] = [i.dxf.layer for i in entities]
    # A known issue with converting DWG files to DXF is that some elements have their x coordinate reversed
    if len(output.x[output.x <= 0]) != 0:
        print("Negative x values! This is a known issue with dxf files")
        print("Mirroring negative x values")
        output.x = np.abs(output.x)
    # Apply scaling factor
    output = output.apply(lambda x: x * scaling if x.name in ['x', 'y'] else x)
    return output
```

The second function builds the query and passes it to the first.

```python
def extract(model, layer=None, block=None, scaling = 0.001):
    """
    Extracts all objects of a certain block type within a specified layer of a dxf file.

    Parameters
    ----------
    model : ezdxf modelspace object
        A modelspace object created from a loaded dxf file. 
        See the ezdxf help file for 'readfile' and 'modelspace' methods
        
    layer : string
        A string specifying the layer name in the modelspace object
        
    block : string
        A string specifying the block name in the modelspace object
        
    scaling : positive real value
        Depending on the units of the modelspace and the desired units, a scaling
        may be preferred. The default is a unit of mm and a desired unit of metre.
         (Default value = 0.001)

    Returns
    Pandas dataframe object with ID, x and y coordinate values. One row per object.
    -------

    """
    if layer is None:
        query_string = '*[name==\"%s\"]' % block
    elif block is None:
        query_string = '*[layer==\"%s\"]' % layer
    else:
        query_string = '*[layer==\"%s\" & name==\"%s\"]' % (layer, block)
    output = extract_query(model, query_string, scaling)
    return output
```

Note that the default scaling is 0.01. This is because typical CAD files dealing with building drawings have units of cm, and I would like to store x,y positions in metres.

## Taking it further

At [Arcadis Gen](https://arcadisgen.com/) we take a package based approach to consultancy to make enduring products from one-off engagements and shorten the development cycle for future similar projects. To do this I developed a Python package called `cadextract`, built on top of ezdxf and providing helper functions to extract seat plans. The functions in this package look a little similar to the above, but also include `fuzzy_extract` to deal with queries using regular expressions, `batch_extract` that extracts from multiple DXF files using the same query and stores them in a single dataframe, plotting objects, and more.

In the third and final part of this series on CAD I'll take a more speculative look at where you might be able to take this programmatic approach, and where future opportunities might lie.

