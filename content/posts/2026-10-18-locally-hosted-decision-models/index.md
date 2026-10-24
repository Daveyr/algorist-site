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

A few months ago I deployed an [LLM on an old smartphone](/posts/2026-06-03-ai-llama-server-on-a-phone). At the time I said that I was going to follow up later to share my experiences with various models and use cases and so here I am. What I didn't know back in June is that decision models would appear. They represent a fantastic fit for a frugal phone like mine, and a good first use case to share.

## What are decision models?

The first decision model I heard of was [Jev](https://typesafe.ai/blog/introducing-system-one-models-and-jev). Jev and other decision models are a different kind of model from a pure language model. Instead of generating text one token at a time, they read a state once and *score* a set of typed questions against criteria that you define in advance, returning type-safe structured answers (*choice*, *score* or *noul*) with a calibrated probability attached to each one, plus a confidence score for *choice* and *score* answers. As the answer is always one of the options you supplied, the model cannot hallucinate a schema (although that doesn't mean it always chooses the correct answer), and there is nothing to parse or validate before your code uses it. Every question in a request is answered in parallel in a single forward pass, so a decision comes back in milliseconds rather than the seconds a chat model takes, and the models themselves are small enough to run on hardware you would never try to run an LLM on (like my phone). TypeSafe report Jev as matching frontier LLMs on these tasks while being two orders of magnitude faster and cheaper, and the open models are tiny by comparison (hundreds of millions of parameters not hundreds of billions for frontier LLMs).

There's a richness in the terminology used around Jev and so I thought I'd restate it here in case you miss it.

* Jev is so named in honour of William Stanley Jevons, a 19th century economist who defined what we know as the _Jevons Paradox_. It refers to the paradox of growing efficiency of coal use in plant and machinery through the industrial revolution and how this led to *more coal use overall*. In the same way, Typesafe is inferring that as we make AI use more efficient (for instance, through lightweight decision models), overall AI consumption will still grow as we find even more use cases for AI at lower price points.
* Typesafe's name refers to one of the key strengths of decision models: the model will never hallucinate the type of its response (choice, score, noul). This is known as being type safe.
* Even the endpoint has rich meaning. Decision models have converged on Typesafe's endpoint, which is `/v1/systemone`. System One refers to one of two systems of the brain, first posited by Daniel Kahneman and Amos Tversky in their ground breaking book, Thinking Fast and Slow. I love this book and recommend you read it. Where System Two is the slower reasoning part of our brain, System One is the much faster, instinctive, reactive survivor part of our brain. You can see why Typesafe chose this nomenclature for Jev.

## Motivations.

So what are decision models good for? When I first heard of them, my immediate thought ran to these examples from my work:
* Asset condition grading on a scale (1-5, very poor to very good, etc.) from a free text field based on strict criteria.
* More specifically, sewer pipe condition categorisation (MSCC system in the UK) from CCTV images (potentially using an image-text to text model like [Clef](https://huggingface.co/Cloudflare/clef)).
* Risk classification (typically 1-5 across both likelihood and consequence) from corporate risk registers.
* Judging whether an asset has truly failed from the work order data (i.e., whether it constitutes a fatality in Weibull modelling or mean-time-to-failure terminology). 
* Failure mode classification for FMEA or even FMECA analysis, which includes criticality treatment.
* Feature engineering from free text fields (all too common in source systems) to enhance predictive models.

## Prerequisites
If you have been following along from [before](/projects/2026-06-08-llama-server-phone-guide/), you need to make sure your build is recent enough. The `/v1/systemone` endpoint arrived in llama.cpp v0.6.0 (October 2026, PR [#29818](https://github.com/ggml-org/llama.cpp/pull/29818)), so if you followed the June guide at the time you need to pull and rebuild.

```bash
cd ~/llama.cpp
git pull
cmake --build build -j4
```

The above reuses the same `build` directory I showed in the walkthrough, so it involves 10–30 minutes on the phone and nothing else changes. The rest carries over: Termux, SSH, the `opencode.json` provider block.

>[!TIP]
Make your life easier by installing ccache (type "pkg ccache" in Termux), which speeds up the build process on old hardware and bad internet connections through caching.

## Model options

There are a lot of decision models already out there! Fortunately there are a healthy subset that are compatible with my existing phone setup. These include laya, julia-1, lev, openjev, kev, clef and nimble. I went with a very lightweight text only (no image) model, laya. It has only 0.4B parameters and is only around ~450mb in size, even at Q8_0 quantisation.

As I am using llama.cpp, I used the GGUF, not the raw HF repo. convaiinnovations/laya is the official laya model on huggingface but it is a Transformers/ModernBERT model served by its own laya-serve Python server. The llama.cpp-native build is ggml-org/Laya-GGUF (ModernBERT encoder compiled into ggml). That's the one that plugs into /v1/systemone.

## Install and test

From my phone I run:

```bash
llama-server \
  -hf ggml-org/Laya-GGUF:Q8_0 \
  -c 512 -t 4 -fit off \
  --host 0.0.0.0 --port 8080
```

And that's it. Next, I tested the model from another computer on the same network, using two sewer pipe reaches from a CCTV survey. Each request asks two questions about the same state: a choice between three condition grades, each described by criteria, and a noul asking whether the asset needs intervention now. First, the fair reach:

```bash
curl -s -m 30 http://192.168.1.210:8080/v1/systemone -H 'Content-Type: application/json' -d \
'{"state":"CCTV survey of a 150 mm vitrified clay pipe, reach MH14 to MH15. Hairline crack near the crown, about 1 m long, no displacement. Light surface wear along the invert with some silt. Joints intact, no infiltration or root ingress. Pipe remains fully functional.",
"questions":{"condition":{"type":"choice","instructions":"What is the condition grade of this asset?",
"criteria":{"poor":"Structural or functional defects, such as fractures, spalling, open or displaced joints, root ingress, active infiltration or lost section.",
"fair":"Minor defects to monitor, such as hairline cracks without displacement, surface wear or silt. No infiltration or structural damage.",
"good":"No defects, or only cosmetic marks. Joints intact and fully functional."}},
"intervene":{"type":"noul","instructions":"Does this asset need intervention now?"}}}'
```

Second, the poor reach. Only the state changes:

```bash
curl -s -m 30 http://192.168.1.210:8080/v1/systemone -H 'Content-Type: application/json' -d \
'{"state":"CCTV survey of a 150 mm vitrified clay pipe, reach MH22 to MH23. Crown fractured over 2 m with spalled material. Open joint at 6 m with root ingress. Active infiltration at two joints. Invert has lost about a third of its section. Capacity is reduced and collapse risk is high.",
"questions":{"condition":{"type":"choice","instructions":"What is the condition grade of this asset?",
"criteria":{"poor":"Structural or functional defects, such as fractures, spalling, open or displaced joints, root ingress, active infiltration or lost section.",
"fair":"Minor defects to monitor, such as hairline cracks without displacement, surface wear or silt. No infiltration or structural damage.",
"good":"No defects, or only cosmetic marks. Joints intact and fully functional."}},
"intervene":{"type":"noul","instructions":"Does this asset need intervention now?"}}}'
```

This returns, for the fair reach:

```json
{
  "model": "ggml-org/Laya-GGUF:Q8_0",
  "answers": {
    "condition": {
      "type": "choice",
      "choice": "good",
      "probabilities": {
        "poor": 0.1824619558584677,
        "fair": 0.2557620965808609,
        "good": 0.5617759475606714
      },
      "confidence": 0.3426639213410071
    },
    "intervene": {
      "type": "noul",
      "noul": 0.027341530510618356
    }
  },
  "usage": {
    "input_tokens": 249,
    "output_tokens": 0
  }
}
```

and for the poor reach:

```json
{
  "model": "ggml-org/Laya-GGUF:Q8_0",
  "answers": {
    "condition": {
      "type": "choice",
      "choice": "poor",
      "probabilities": {
        "poor": 0.8939986689867528,
        "fair": 0.05296165417230909,
        "good": 0.053039676840937994
      },
      "confidence": 0.8409980034801291
    },
    "intervene": {
      "type": "noul",
      "noul": 0.4689226425618783
    }
  },
  "usage": {
    "input_tokens": 257,
    "output_tokens": 0
  }
}
```


The test succeeded. Laya on the phone's local IP address (192.168.1.210:8080) answers both requests in the expected format, and the poor reach comes back as poor. The fair reach doesn't: Laya's top grade is good, with fair second. These are single runs to show the format, not a measure of accuracy, and running them again gave the same numbers. Let's dissect the fair reach first, since it's the one Laya disagreed with.

For the fair reach:

- condition → good with probability 0.562 (confidence 0.343). fair comes second at 0.256 and poor third at 0.182. Under the fair criteria, a hairline crack without displacement is a defect to monitor, so I expected fair.
- The confidence is (3 × 0.562 − 1) ÷ 2 = 0.343. With three options, [TypeSafe's confidence](https://docs.typesafe.ai/confidence) comes from how the probability is spread, not from a separate estimate. This spread is wide, so the answer isn't decisive and I wouldn't act on it automatically.
- One possible cause, which I haven't tested: the closing sentences use "joints intact" and "fully functional", the same words the good criterion uses.
- intervene → 0.027 (noul). A noul is the probability of yes and has no confidence of its own. A low value fits a reach to monitor.
- input_tokens: 249, output_tokens: 0. The criteria and questions count as input too, so the prompt is longer than the old ticket's 111 tokens. Output is still zero, because the model scores the options rather than generating text.

For the poor reach:

- condition → poor with probability 0.894 (confidence 0.841). fair and good each get 0.053.
- The confidence is (3 × 0.894 − 1) ÷ 2 = 0.841. The probability sits on one grade, so the answer is decisive.
- intervene → 0.469 (noul). TypeSafe's suggested starting thresholds are 0.7 for yes and 0.3 for no, so this sits in between. The model is confident about the grade but not about the action, so I wouldn't act on it automatically.
- input_tokens: 257, output_tokens: 0. Both requests ask the same questions, so the 8-token difference comes from the two state texts.
- The model id reported is ggml-org/Laya-GGUF:Q8_0 in both runs, confirming the local model is loaded and used.

Two runs can't show how often Laya gets a grade right. That's what the evals in the next steps are for, and this pair shows why they're needed: the poor reach came back poor, but the fair reach came back good.

## First impressions

## Next steps

It's exciting to have a real-world productive use case for my AI-on-a-phone experiment but there's a few things still to do before I can put it to use.

1. Registering as a model in OpenCode. I suspect that due to the unusual endpoint it will need to be invoked via an MCP tool, potentially wrapped in an agent that can parse the question, criteria and the response type.
1. Evals. So far I haven't seen any evaluations tailored to asset-intensive industrial use cases so I'd like to explore that further. laya seems not to do well in standard benchmarks and so having a ranking of small decision models against an industry benchmark might be useful for myself and others to choose the best model amongst a burgeoning set of contenders.
