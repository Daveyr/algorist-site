---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Gitlab Runner"
summary: "Gitlab runner in a docker container. Compatible with Raspberry Pi"
authors: ["Richard Davey"]
tags: ["Docker", "Gitlab", "Git", "Raspberry Pi"]
categories: 
date: 2020-02-06T10:43:00

# Optional external URL for project (replaces project detail page).
#external_link: "https://github.com/Daveyr/gitlab-runner"

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Left"
  preview_only: false

# Custom links (optional).
#   Uncomment and edit lines below to show custom links.
# links:
# - name: Follow
#   url: https://twitter.com
#   icon_pack: fab
#   icon: twitter

url_code: "https://github.com/Daveyr/gitlab-runner"
url_pdf: ""
url_slides: ""
url_video: ""

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: ""
---


# README

<img src="featured.jpg" alt="Gitlab logo"     style="display: block; 
           margin-left: auto;
           margin-right: auto;
           width: 30%;"/>

A runner is a server that is able to execute a CI/CD pipeline. These pipelines contain one or more jobs that check a version controlled repository for build errors, code style, etc. This repository contains a dockerfile and bash scripts to make a Gitlab runner. It is intended to be used with a Gitlab repository that has a CI/CD pipeline configured according to a `.gitlab-ci.yml` file in the parent directory. 

As long as the base image in your .gitlab-ci.yml file is compatible with arm architecture, this runner will work on a Raspberry Pi. If the base image isn't compatible then you will have the same experience as [this](https://www.talvbansal.me/blog/maximising-gitlab-ci-s-free-tier/). Of course, you can build this image on an x86 machine and it will work with most base images a CI file is likely to contain.

## To build

Instructions from https://www.devils-heaven.com/gitlab-runner-finally-use-your-raspberry-pi/. Note that you should copy the token that is located on your gitlab account, found under the project menu:
`Settings> CI / CD> Runners>Specific Runners`

```
docker build -t gitlab-runner --build-arg token=<TOKEN> .
```

## To run

From https://itnext.io/docker-in-docker-521958d34efd. Note we need to mount a volume as we run the container. This is because the runner depends on Docker. Running Docker in a Docker container is tricky so the easiest method is to "borrow" the host Docker service instead, which we do by mounting its .sock file.

```
docker run -v /var/run/docker.sock:/var/run/docker.sock gitlab-runner
```
