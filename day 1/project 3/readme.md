# P3 – Simple Evaluations

## Overview

Simple workflow to **demo n8n’s Evaluation nodes**: it pulls rows from a dataset (users upload `evals.csv` into an n8n Data Table), runs an LLM classification, then records metrics + outputs back to the evaluation run.  

<img width="683" height="269" alt="image" src="https://github.com/user-attachments/assets/53d0a1d2-e6e4-429c-8aea-463f02af531c" />

## Setup

* In n8n, create a **Data Table** (e.g. “evals”) and **upload `evals.csv`** as the dataset. 

## Workflow

1. Import the n8n Eval Workflow from [here](https://github.com/tobiaszwingmann/n8n-ai-bootcamp/blob/main/day%201/project%203/n8n/P3%20%E2%80%93%20Simple%20Eval.json)
2. **Run the workflow** from the Evaluation table ("When fetching a dataset") node.
3. Select **Evaluations** from the top and select **Run Evaluation**
   * Things to try out: 
   		- Modify the system prompt.
   		- Try a different model.
