# P2 – Connect AI model

## Overview

Minimal workflow to **authenticate an AI chat model** (Google Gemini) and **prove it works** by running a simple prompt. 

## Steps

1. **When clicking ‘Execute workflow’** (Manual Trigger)

   * Starts the test run.

2. **Google Gemini Chat Model**

   * Model: `models/gemini-3-flash-preview`
   * Uses stored credentials: **Google Gemini(PaLM) Api account** (authentication check).

3. **Basic LLM Chain**

   * System message: *“You are a pirate”*
   * Prompt: *“Tell me a joke”*
   * If it returns a response, the connection + auth are working.
