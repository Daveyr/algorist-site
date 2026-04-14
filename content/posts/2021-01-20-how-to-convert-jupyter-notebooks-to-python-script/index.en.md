---
title: How to convert Jupyter notebooks to python script
author: ''
date: '2021-01-20'
slug: how-to-convert-jupyter-notebooks-to-python-script
categories:
  - Guide
tags:
  - Python
  - Jupyter
subtitle: ''
summary: ''
authors: []
lastmod: '2021-01-20T14:41:52Z'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Jupyter notebooks are great for experimentation, reporting and sharing. In a project there are often times when you need to transition from this activity to production ready code. The easiest first step is to convert your notebook to a script. In order to do this you will need `jupyter` and `nbconvert` packages installed.

From a terminal or command prompt type,
```
pip install jupyter
pip install nbconvert
```

## Convert a single notebook

To convert a single notebook, just type the following commands in a terminal where the current directory is the location of the file. Jupyter will convert the notebook to a script file with the same name but with file ending `.py`.

```
jupyter nbconvert --to script notebook.ipynb
```

## Convert multiple notebooks

To convert multiple notebooks simultaneously only requires the typing of additional filenames.

```
jupyter nbconvert --to script notebook1.ipynb notebook2.ipynb
```

You can even use wildcard searches to make things easier. For instance,

```
jupyter nbconvert --to script notebook*.ipynb
```