---
title: 'Extract features from CAD files Part 3: The future'
author: ''
date: '2021-09-28'
slug: extract-features-from-cad-files-part-3-the-future
categories:
  - Blog
tags:
  - Built Environment
  - CAD
  - Python
subtitle: ''
summary: ''
authors: []
lastmod: '2021-09-28T16:46:59+01:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

In this third part of the series of posts on extracting objects from a CAD document, I’ll discuss how you might develop this CAD extraction tool further and the problems you might be able to solve.

## Prologue
In [Part 1](https://www.algorist.co.uk/post/extract-features-from-cad-documents-part-1-a-primer/) we looked at the structure of a CAD file and built up a strategy to extract seat types and locations from an architect's floor plan. The motivation for this is to provide seat location data to a model that creates a stack plan with optimal locations of teams to office amenities and other teams they collaborate with.

In [Part 2](https://www.algorist.co.uk/post/extract-features-from-cad-documents-part-2-using-ezdxf/) we built an extraction tool based on the Python `ezdxf` package that can read and query DXF files. We loaded a floor plan in DXF, printed the block types in a layer, extracted all block inserts that matched a layer and/or block type query and outputted them to a pandas dataframe.

## Additional feature extraction

As long as a CAD drawing is segregated by block type and/or layer, several floor features can be extracted with their own query and appended to the dataframe of floor features. Currently only assignable seating has been extracted from a single layer, but what else might you want to extract?

### Floor features a team may need

Often teams will use meeting rooms. Some teams use meeting rooms more than others, or need more meeting rooms simultaneously. Large teams will typically require larger meeting rooms compared with small teams. The typical meeting room layout is to have a single table in the centre of the room, which means that running a query for, e.g., "small conference table" and "large conference table" will return you the count and the centre point of every small and large conference room, respectively. Alternatively, if the seats in a meeting room are stored on a layer separate to assignable desk seats then this could be a way of tracking the location and seating capacity of a meeting room.

A similar approach could be used to extract printers, breakout rooms, kitchen facilities and everything else a team may have a preference for. It could even be used to track accessibility requirements (e.g., proximity to a lift, disabled toilets, etc.).

### Neighbourhoods/zoning

A related - but more complicated - concept is to define neighbourhoods or zones within a floor. Does a team need quiet space away from stairwells, kitchens and thoroughfares? What about key swipe access to secured areas on a floor? As long as this is defined in an appropriately named block or layer then we can extract it, calculating the spatial overlap with seats in that zone using a Python function.

## Drawing/export

`ezdxf` has addon modules, one of which deals with drawing and export of CAD files. Below is a very simple example of how to read a DXF file and render as a PNG image file to disk.

```python
from ezdxf import recover
from ezdxf.addons.drawing import matplotlib

# Exception handling left out for compactness:
doc, auditor = recover.readfile('your.dxf')
if not auditor.has_errors:
    matplotlib.qsave(doc.modelspace(), 'your.png') 
```

## Streamline the use of ODA Converter

In part 1 we discussed the need to convert a CAD file from proprietary DWG format to DXF format using the open source file converter provided by the [Open Design Alliance](https://www.opendesign.com/guestfiles/oda_file_converter). Instead of opening this software separately we can call it from `ezdxf` using another addon.

```python
from ezdxf.addons import odafc

# Load a DWG file
doc = odafc.readfile('my.dwg')

# Use loaded document like any other ezdxf document
print(f'Document loaded as DXF version: {doc.dxfversion}.')
msp = doc.modelspace()
...

# Export document as DWG file for AutoCAD R2018
odafc.export_dwg(doc, 'my_R2018.dwg', version='R2018')
```

## Generative design	

The ultimate aim of the stack plan modelling tool is to develop it into a product. What better way to do this than to create a workflow that starts with a CAD drawing and ends with an annotated CAD drawing labelled according to team locations? You could reassign seats from your seats layer to new layers named after the relevant teams on that floor. However, the more appropriate method is likely to use attribute definitions.

Below is an example of isolating an entity and changing an attribute. For a real use case, `entities` would be a list of block IDs of seats that belong to a certain team and the attribute to change would be the team ID.

```python
doc = ezdxf.readfile(filepath)
model = doc.modelspace()

# load a data frame with unique ID that refers to a block insert "handle", and a team name
# define a query_string to search for the appropriate layer/blocks that refer to your allocatable seats
entities = [x for x in model.query(query_string) if x.has_dxf_attrib('name')]

if len(entities):
    entity = entities[0]  # process first entity found
    for attrib in entity.attribs:
        if attrib.dxf.tag == "diameter":  # identify attribute by tag
            attrib.dxf.text = "17mm"  # change attribute content

```

## Stretch goal

In practice a client will take a stack plan recommendation and remodel a floor to include additional teams, additional seats (if the recommended teams for a floor mean the floor is over capacity) or to space desks out if the floor is under capacity. Wouldn't it be great if we could generate floor plans automatically, or at least edit an existing one to accommodate these changes? Between a model that can learn floor plan design principles (e.g., minimum separation between objects, coincidence of a seat with a desk, etc.) and the ability to insert new block references into the model (see below), we would have all we need to achieve this.

An example from the [ezdxf](https://ezdxf.readthedocs.io/en/stable/tutorials/blocks.html?highlight=random#block-references-insert) documentation.

```python
import ezdxf
import random
def get_random_point():
    """Returns random x, y coordinates."""
    x = random.randint(-100, 100)
    y = random.randint(-100, 100)
    return x, y
# Get the modelspace of the drawing.
msp = doc.modelspace()

# Get 50 random placing points.
placing_points = [get_random_point() for _ in range(50)]

for point in placing_points:
    # Every flag has a different scaling and a rotation of -15 deg.
    random_scale = 0.5 + random.random() * 2.0
    # Add a block reference to the block named 'FLAG' at the coordinates 'point'.
    msp.add_blockref('FLAG', point, dxfattribs={
        'xscale': random_scale,
        'yscale': random_scale,
        'rotation': -15
    })

# Save the drawing.
doc.saveas("blockref_tutorial.dxf")
```