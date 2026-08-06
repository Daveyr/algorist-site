---
title: How to set Microsoft Foundry as a provider in OpenCode and Copilot CLI
author: ''
date: '2026-08-08'
slug: []
categories: []
tags:
  - AI
  - Tutorial
draft: true
---

## Be your own AI provider

After Microsoft Copilot's [move from subscription requests to credit-based usage](<insert url here>) on 1 June 2026, the company has tried to lighten the bill shock by providing access to cheaper open weight models, such as Kimi K2.7 Code (available just over a one month [later]()https://github.blog/changelog/2026-08-06-kimi-k3-is-now-available-in-github-copilot/). Meanwhile, there have been plenty of even cheaper model options on _another_ Microsoft enterprise platform hiding in plain sight: **Microsoft Foundry**. Read on to learn how to consume models you deploy yourself on  MS Foundry alongside your regular options in both OpenCode and Copilot CLI.

## Prerequisites

This is not a tutorial on Microsoft Foundry. The new Foundry view is pretty intuitive, documentation from Microsoft is pretty good for Foundry (in my opinion), and the barrier to entry to do the basics is pretty low (just don't try and do evaluations straight away - they are a nightmare).

To follow along, you will need to have:
* A Foundry azure resource group (called a "project")
* A base model deployment in that project, with an endpoint and API key
* Azure model read access for your user ID

When you look at the model endpoint, note how it conforms to the common openAI API endpoint format. We used it previously to [connect OpenCode Go to openwebUI](<insert blog url here>) and we're using it again to connect Foundry to OpenCode/Copilot. Foundry does add a small quirk though. If your project is called "foundry-project":

* The provider will be called "foundry-project",
* The base URL will be `https://foundry-project` `-resource` `.services.ai.azure.com/` `openai/v1`

I've separated the parts of the URL to show the first part (the project name) and the last part (the openAI compatible url ending) are what we expect, but the middle (appending "-resource" to the project name and the top level Azure domain) are something you couldn't guess.

## OpenCode

We'll start with OpenCode first as the configuration is a lot easier. OpenCode's main USP is that it connects to everything, and Foundry is as equally well [documented](<insert opencode document reference to MS Foundry provider config>) as any other provider.

Foundry provider, model definition

>[!TIP]
>For business users and privacy conscious individuals, having free Zen models in OpenCode is more of a liability than a bonus. Using the _enabled_providers_ key with an array containing your providers allows you to only display allowed providers. See below for an example.

```
{
  "$schema": "https://opencode.ai/config.json",
  "enabled_providers": [
    "github-copilot",
    "foundry-project"
  ],
  "model": "foundry-project/DeepSeek-V4-Flash",
  "provider": {
    "foundry-project": {
      "npm": "@ai-sdk/openai-compatible",
      "name": "Solution Delivery Foundry",
      "options": {
        "baseURL": "https://foundry-project-resource.services.ai.azure.com/openai/v1"
      },
      "models": {
        "DeepSeek-V4-Flash": {
          "name": "DeepSeek V4 Flash"
        },
            "Kimi-K2.6": {
          "name": "Kimi-K2.6"
        }
      }
    }
  }
}
```

## Copilot

Assuming you have installed Copilot CLI in Windows Subsystem for Linux (WSL) or in Linux, edit or create your ~/.bashrc file with the following.

```
alias copilot-foundry='export COPILOT_PROVIDER_BASE_URL=https://foundry-project-resource.services.ai.azure.com/openai/v1 && \
export COPILOT_PROVIDER_API_KEY=<KEY HERE> && \
export COPILOT_MODEL=DeepSeek-V4-Flash && \
export COPILOT_PROVIDER_MAX_PROMPT_TOKENS=1000000 && \
export COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000 && \
copilot'
```
### Copilot is not as friendly

* Copilot CLI is often enterprised managed and is intended to have only one centrally managed provider list
* You must explicitly declare each model as well as its provider. Good luck if you are hosting many deployed models.

## Foundry VSCode chat provider

All the limitations on declaring models and their parameters apply to VS Code. However, you _are_ able to specify custom endpoints rather than having to resort to workarounds with aliases.

Below is the same example Foundry custom provider as above, but this time we add another model definition (Kimi-K2.6)

```
      {
            "name": "foundry-project",
            "vendor": "customendpoint",
            "apiKey": "${input:chat.lm.secret.-28e8f8fd}",
            "apiType": "chat-completions",
            "models": [
                  {
                        "id": "DeepSeek-V4-Flash",
                        "name": "DeepSeek-V4-Flash",
                        "url": "https://foundry-project-resource.services.ai.azure.com/openai/v1",
                        "toolCalling": true,
                        "vision": true,
                        "maxInputTokens": 1000000,
                        "maxOutputTokens": 128000
                  },
                  {
                        "id": "Kimi-K2.6",
                        "name": "Kimi-K2.6",
                        "url": "https://foundry-project-resource.services.ai.azure.com/openai/v1",
                        "toolCalling": true,
                        "vision": true,
                        "maxInputTokens": 128000,
                        "maxOutputTokens": 16000
                  }
            ]
      }
]
```