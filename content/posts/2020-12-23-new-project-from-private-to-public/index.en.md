---
title: 'New project: from private to public'
author: ''
date: '2020-12-23'
slug: new-project-from-private-to-public
categories:
  - Blog
tags:
  - Python
  - Raspberry Pi
  - IoT
subtitle: ''
summary: ''
authors: []
lastmod: '2020-12-23T15:21:38Z'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---

Earlier in the year I wrote about using [systemd](https://www.algorist.co.uk/post/resilient-systemd/) in the context of setting up a service for temperature logging and visualisation on a Raspberry Pi. Whilst the project has been stable for a long time, I'm now releasing it publically. [Here](https://www.algorist.co.uk/project/internal-project/pi_temperature/) it is. Key features include:

* Logs temperature using a Raspberry Pi and DS18B 1-wire device
* Queries the Dark Sky weather API 
* Writes temperature readings and weather data to a local database
* Streams temperature readings and weather data to Adafruit feeds, which can be used to build a dashboard

Hopefully the instructions are easy to use and the various features can be turned off as needed

## From private to public
In many ways keeping a private git repository can foster bad habits. My delay to make it public was mainly due to hard coding API keys: fine for local use but a liability in code meant for sharing. In the end I worked out how to separate my private keys from the code but by the time that happened they were in the git history for all the world to see.

### Scrubbing git history
It is possible to selectively scrub git history. This Stack Overflow [post](https://stackoverflow.com/questions/872565/remove-sensitive-files-and-their-commits-from-git-history) details how to purge a file from git history; the github [faq](https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/removing-sensitive-data-from-a-repository) does the same. However, it gets a little messy when the file in question is the main file containing code within the repository.

In the end the benefit of retaining a git history without my main file didn't outweigh the cost of going through the steps and risking either corrupting the repo or revealing the keys. I just made a separate public repo.

## Hiding keys
In the same spirit, I opted for a simple approach to hide the keys. I wrote a file called `api.txt` and populated each line with a key:value pair. I then added the filename to `.gitignore` to ensure I didn't add it to a commit, thereby nullifying the whole point of the exercise. I then used the code below to load it into a dictionary object within python.

``` python
# Test for reading api key text file

from os import listdir
from os.path import isfile, join
print(listdir())

api = {}
with open('api.txt', 'r') as file:
    for line in file:
        (key, value) = line.split()
        api[key] = value
file.close()
```

## A better way?
Both R and Python understand environment variables. For R we can use the `.Renviron` file, populating it with key:value pairs such as `MY_KEY=12345`. We can recall this variable in code using `Sys.getenv("MY_KEY")`.

For Python we can do a similar thing in a few different ways. Anaconda has its own method; there's also decouple:

```python
from decouple import config

API_KEY  = config('MY_KEY')
```

The only difference with the R method is that the file that stores the keys should be called `.env`.

Another method would be to use the `os` package to set and get variables. Obviously the setting of variables would happen in a script not added to the repository.

``` python
import os

# Set environment variables
os.environ['MY_KEY'] = '12345'

# Get environment variables
KEY = os.getenv('MY_KEY')
```