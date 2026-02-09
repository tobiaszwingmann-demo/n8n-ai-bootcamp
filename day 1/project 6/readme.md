# P6 – Workflow Logic

**Tip:** Import this workflow directly into n8n and ensure Google Gemini credentials are configured.

---

## Workflow Overview

**Purpose**
This workflow demonstrates **AI-driven workflow logic** for an IT support chat system.
Incoming chat messages are automatically classified and either:

* handled directly by an AI chatbot, or
* escalated into a structured IT support ticket for human follow-up.

All reasoning, routing, and classification is performed using **Gemini 2.5 chat models**.

**Trigger**

* Chat message (Chat Trigger)

---

## High-Level Flow

1. User sends a chat message
2. AI **Router** decides whether the issue can be automated
3. If *Automated* → Chatbot responds directly
4. If *Route to IT* → Ticket is created, prioritized, and stored
5. User receives a confirmation message with a ticket ID

---

## Nodes Used

1. **When chat message received**
2. **Router (LLM Chain)**

   * Google Gemini Chat Model
   * Structured Output Parser
3. **If**
4. **Chatbot (Agent)**

   * Google Gemini Chat Model
   * Simple Memory
5. **Ticket Writer (Agent)**

   * Google Gemini Chat Model
   * Structured Output Parser
   * Simple Memory
6. **Edit Fields**
7. **Urgency (LLM Chain)**

   * Google Gemini Chat Model
   * Structured Output Parser
8. **Create a file (GitHub)**
9. **Edit Fields (final response)**

---

## Node-by-Node Explanation

---

### 1. When chat message received

**Type:** Chat Trigger

**Purpose:**
Starts the workflow whenever a user sends a chat message.

* Public chat endpoint
* Greets the user and asks for their name
* Serves as the single entry point for the workflow

---

### 2. Router

**Type:** LLM Chain (Gemini 2.5)

**Purpose:**
Decides whether the request can be handled automatically or must be routed to human IT support.

The model classifies each request as one of:

* `Automated`
* `Route to IT`

A **Structured Output Parser** enforces a predictable output format:

```json
{ "prio": "Automated" }
```

This node is the **decision engine** of the workflow.

---

### 3. If

**Type:** If node

**Purpose:**
Routes execution based on the router’s decision:

* `Automated` → Chatbot
* anything else → Ticket creation path

---

### 4. Chatbot

**Type:** Agent (Gemini 2.5)

**Purpose:**
Handles simple IT support questions automatically.

* Uses a system prompt defining an IT helpdesk role
* Can ask clarifying questions
* Provides step-by-step troubleshooting
* Uses **Simple Memory** to maintain short conversation context

This path ends once the user’s issue is resolved.

---

### 5. Ticket Writer

**Type:** Agent (Gemini 2.5)

**Purpose:**
Creates a structured IT support ticket when automation is not appropriate.

The agent extracts:

* User name
* Short issue description
* A friendly response confirming escalation

A **Structured Output Parser** ensures consistent ticket fields.

---

### 6. Edit Fields

**Type:** Set

**Purpose:**
Generates a unique ticket ID using the current timestamp.

This ID is reused in:

* the ticket file
* the final user response

---

### 7. Urgency

**Type:** LLM Chain (Gemini 2.5)

**Purpose:**
Classifies the ticket as:

* `Urgent`
* `Not urgent`

Based on business-impact rules (outages, security, critical work blockers, etc.).

The result is parsed into structured output for reliable downstream use.

---

### 8. Create a file

**Type:** GitHub

**Purpose:**
Creates a ticket file in a GitHub repository containing:

* Priority
* User name
* Submission time
* Issue description

This simulates handing the ticket off to a real IT system.

---

### 9. Edit Fields (final response)

**Type:** Set

**Purpose:**
Builds the final chat message sent back to the user:

* Confirms ticket creation
* Displays the ticket ID
* Politely closes the conversation

---

## What This Demo Shows

* AI-based routing and decision-making
* Multiple Gemini 2.5 models working together
* Structured outputs for reliable workflow logic
* Combining chat, automation, and human escalation in one flow

---

## Example Inputs to Try

* “I can’t log in to Outlook”
* “My laptop fell down and is broken”
* “I need admin rights to install software”
* “Excel won’t open any files”