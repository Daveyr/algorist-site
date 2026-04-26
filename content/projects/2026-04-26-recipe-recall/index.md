---
title: Recipe Recall
author: Richard Davey
date: '2026-04-26'
slug: []
summary: 'Python recipe scraper exposed as an MCP tool'
categories:
  - Web scraper
  - MCP server
tags:
  - Python
  - GitHub
  - AI
---

This was the result of a one-hour AI hackathon organised by [Steel-AI](https://alex-kelly.blog/Sheffield_event_info/event_2_advert_agents.html) (not including the 3 hours of debugging and documentation afterwards!).

Don't you just hate the time it takes to write a food shopping list every week to make the recipes you want to make? Use this project if you want hallucination-free ingredients lists from a reliable source called directly from your preferred LLM. Also refer to this for inspiration for your own agentic AI tools configured using a locally hosted MCP server.

Python scraper and fastmcp script built with help from GPT-5 mini, switching to Claude Haiku 4.5 later on (which also helped build the first draft of this readme). MCP server tested on both Qwen3:8b and Llama 3.2:3b locally hosted via Ollama.

## Implementation summary



1. bbcgoodfood_scraper.py

    * Standalone Python script that searches BBC Good Food recipes
    * Extracts ingredients from ul.ingredients-list.list elements
    * Parses ingredients into quantity/unit/ingredient_name components
    * Handles text formatting quirks (e.g., "50gdried" → "50g dried")
    * Fully functional when run directly

1. fastmcp_quickstart.py

    * MCP server exposing scraper and utilities as callable tools
    * Tools: add_note, add, test_network, search_recipes
    * Uses stdio transport for subprocess communication
    * File logging to recipe_recall.log with timestamps (on the function-in-tool branch)

