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

Below is a minimal `opencode.json` configuration file that does the following:

* Restricts the providers to Copilot and a custom provider I call 'foundry-project'
* Define the custom foundry-project url
* Define two models from the provider (Deepseek V4 Flash and Kimi K2.6)
* Set the default model to be Deepseek V4 Flash

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

Where OpenCode is open, agnostic to provider and the issue is locking down providers you _don't need_ (looking at you, privacy sucking free Zen models), Copilot is the inverse. Copilot has the feel of an enterprise tool: providers are often centrally managed along with the models they serve. To add a custom provider you may need to supply environment variables at runtime.

Assuming you have installed Copilot CLI in Windows Subsystem for Linux (WSL) or in Linux, edit or create your ~/.bashrc file with the following.

```
alias copilot-foundry='export COPILOT_PROVIDER_BASE_URL=https://foundry-project-resource.services.ai.azure.com/openai/v1 && \
export COPILOT_PROVIDER_API_KEY=<KEY HERE> && \
export COPILOT_MODEL=DeepSeek-V4-Flash && \
export COPILOT_PROVIDER_MAX_PROMPT_TOKENS=1000000 && \
export COPILOT_PROVIDER_MAX_OUTPUT_TOKENS=128000 && \
copilot'
```

After restarting your terminal or typing `source ~/.bashrc`, you can type the alias `copilot-foundry` and launch your Foundry model.

## Foundry VSCode chat provider

Copilot Chat in VS Code is a little easier to work with than Copilot CLI, insofar as you can specify custom endpoints rather than having to resort to workarounds with aliases. However, you do need to specify each model in turn (like OpenCode) along with their individual parameters (unlike OpenCode).

Below is the same example Foundry custom provider as for Copilot, but for the same two models as we had for OpenCode. Note the API key is supplied on the first invocation of the model in the chat window and stored as a secret.

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

## Summary

When I first used this technique, I estimated I had saved around $70 in a week across 4 people and 15 million tokens, comparing Foundry Deepseek against Copilot's default model of GPT-Codex 5.3. Now there are cheaper models available directly from Copilot, the savings per token may not be as great, but there is still nothing as cheap as Deepseek V4 Flash or MiMo 2.5 available on Copilot as of now. 

Maybe things will change again, but at least you know how to connect terminal user interfaces to Microsoft Foundry. And since the base model deployments are all openAI API compatible, it should serve to demystify any other such model deployment process you may stumble across in future.