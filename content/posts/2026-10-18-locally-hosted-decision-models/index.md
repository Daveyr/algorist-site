---
title: Locally hosted decision models
author: ''
date: '2026-10-18'
slug: []
categories:
  - Guide
tags:
  - AI
  - Android
  - Hardware
draft: true
---

A few months ago I deployed an [LLM on an old smartphone](/posts/2026-06-03-ai-llama-server-on-a-phone). At the time I said that I was going to follow up later with my experiences with various models and use cases and so here I am. What I didn't know back in June is that decision models would appear. They represent a fantastic fit for a frugal phone like mine, and a good first use case to share.

## What are decision models?

The first decision model I heard of was [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev). Jev and other decision models are <fill in short explanation about being strong typed, quicker, lower memory requirements>

So what are decision models good for? When I first heard of them, my immediate thought ran to these examples from my work:
* Asset condition grading on a scale (1-5, very poor to very good, etc.) from a free text field based on strict criteria.
* More specifically, sewer pipe condition categorisation (MSCC system in the UK) from CCTV images (potentially using an image-text to text model like [Clef](https://huggingface.co/Cloudflare/clef)).
* Risk classification (typically 1-5 across both likelihood and consequence) from corporate risk registers.
* Judging whether an asset has truly failed from the work order data (i.e., whether it constitutes a fatality in Weibull modelling or mean-time-to-failure terminology). 
* Failure mode classification for FMEA or even FMECA analysis, which includes criticality treatment.
* Feature engineering from free text fields (all too common in source systems) to enhance predictive models.

## Prerequisites
If you have been following along from [before](), you need to make sure <refine these notes: Your build must be recent enough. The /v1/systemone API landed in llama.cpp v0.6.0 (Oct 2026, PR #29818). Your guide is from June 2026, so if you built then you'll need to update:
cd ~/llama.cpp && git pull && cmake --build build -j4>

## Options

There are a lot of decision models already out there! Fortunately there are a healthy subset that are compatible with my existing phone setup. These include laya, julia-1, lev, openjev, kev, clef and nimble. I went with a very lightweight text only (no image) model, laya. It has only 0.4B parameters and is only around ~450mb even at Q8_0 quantisation.

As I am using llama.cpp, I used the GGUF, not the raw HF repo. convaiinnovations/laya is the trending laya model on huggingface but it is a Transformers/ModernBERT model served by its own laya-serve Python server. The llama.cpp-native build is ggml-org/Laya-GGUF (ModernBERT encoder compiled into ggml). That's the one that plugs into /v1/systemone.

## Installing Laya

From my phone I run:

```bash
llama-server \
  -hf ggml-org/Laya-GGUF:Q8_0 \
  -c 512 -t 4 -fit off \
  --host 0.0.0.0 --port 8080
```

After this, I confirmed the model was live and reachable using the following from another computer on the same network.

```bash
curl -s -m 30 http://192.168.1.210:8080/v1/systemone -H 'Content-Type: application/json' -d '{"state":"Customer message: I was charged twice for my order last week and nobody has replied.","questions":{"route":{"type":"choice","instructions":"Which team should handle this?","criteria":{"billing":"payments, charges, refunds, invoices","shipping":"delivery, tracking, lost or late parcels","technical":"bugs, errors, login problems"}},"angry":{"type":"noul","instructions":"Is the customer angry?"}}}'
```


{"model":"ggml-org/Laya-GGUF:Q8_0","answers":{"route":{"type":"choice","choice":"billing","probabilities":{"billing":0.9839928136089258,"shipping":0.008789576661828816,"technical":0.00721760972924533},"confidence":0.9759892204133886},"angry":{"type":"noul","noul":0.7867774868745377}},"usage":{"input_tokens":111,"output_tokens":0}}


The test succeeded. Laya on 192.168.1.210:8080 is responding to /v1/systemone and returned a sensible decision. Let's disect it and learn more about decision models.

- route → billing with probability 0.984 (confidence 0.976) — correct for a double-charge complaint
- angry → 0.787 (noul) — plausible for an unanswered complaint


- input_tokens: 111, output_tokens: 0 — decision models answer via scoring, not generation, so latency should be very low and token cost is just the prompt.
- The model id reported is ggml-org/Laya-GGUF:Q8_0, confirming the native build is loaded.
Natural next steps if you want them:
1. Wire it into OpenCode as a decision subagent, mirroring the guide's opencode.json pattern (point baseURL at http://192.168.1.210:8080/v1), e.g. a triage/routing agent that calls /v1/systemone instead of chat completions.
2. Tune thresholds — pick a confidence cutoff per question (their docs suggest acting above ~0.8 and escalating below).
3. Test edge cases — short/ambiguous tickets, more options (up to 24), and the score question type — to calibrate cutoffs on your real data before relying on it.
Want me to draft an OpenCode agent config or a small client wrapper for the /v1/systemone endpoint?

## First impressions

## Testing and comparisons

## Summary