# P3 – Workflow Logic

**Tip:** Import this workflow directly from the GitHub repository.

## Workflow Overview

**Purpose:** Implement workflow logic that routes IT support chat requests automatically or to human IT support depending on the issue, using LLM classification and chat interaction.

**Trigger:** Chat message (`Chat Trigger`)

**Nodes:**

1. When chat message received
2. Router

   2.1 OpenAI Chat Model

   2.2 Structured Output Parser

3. If
4. Chatbot

   4.1 OpenAI Chat Model

   4.2 Simple Memory
5. Ticket Writer

   5.1 OpenAI Chat Model

   5.2 Structured Output Parser

   5.3 Simple Memory

6. Edit Fields

7. Urgency

   7.1 OpenAI Chat Model

   7.2 Structured Output Parser

8. Create a file

9. Edit Fields1

---

#### Node 1: When chat message received

**Type:** `Chat Trigger (@n8n/n8n-nodes-langchain.chatTrigger)`

**Purpose:** Start the workflow when a chat message is received from a user.

| Parameter             | Value                                                                            |
| --------------------- | -------------------------------------------------------------------------------- |
| **Authentication**    | n8nUserAuth                                                                      |
| **Initial Message**   | Hi there! 👋 Welcome to our IT support. Please tell us your name to get started: |
| **Input Placeholder** | Type your name..                                                                 |
| **Public**            | true                                                                             |

---

#### Node 2: Router

**Type:** `LLM Chain (@n8n/n8n-nodes-langchain.chainLlm)`

**Purpose:** Classify incoming chat messages as *Automated* or *Route to IT* based on complexity and topic.

| Parameter         | Value                                                       |
| ----------------- | ----------------------------------------------------------- |
| **Prompt Type**   | define                                                      |
| **Text Input**    | ```Issue: `{{ $('When chat message received').item.json.chatInput }}` ``` |
| **System Prompt** |                                                             |

```markdown
### **Role**

You are a text classification system for an IT helpdesk.
Your task is to decide whether an incoming IT support request can be handled **automatically** by the chatbot or needs to be **routed to a human IT technician**.

### **Default Behavior:**

Assume **all requests can be automated** unless there are clear indicators that human IT support is required.

### **Classification Labels:**

* **Automated** — The issue is simple, routine, or can likely be resolved using self-service troubleshooting.
* **Route to IT** — The issue is complex, high-risk, or contains keywords/topics suggesting it requires human involvement.

### **Route to IT** if the request includes **Keywords or topics** such as:

   * *hardware*, *broken*, *damaged*, *replacement*, *repair*
   * *network down*, *server*, *outage*, *system crash*
   * *admin rights*, *installation*, *permissions*, *access denied*
   * *security*, *breach*, *virus*, *phishing*, *data loss*
   * *urgent*, *critical*, *can’t work*, *system not starting*
   * *new equipment*, *device setup*, *hardware request*
   * *user demands human assistance* 

### **Otherwise:**

Classify as **Automated** — including typical Windows, Office app, network, or login troubleshooting that follows standard steps.
```

#### Subnode 2.1: OpenAI Chat Model

**Type:** `OpenAI Chat Model (@n8n/n8n-nodes-langchain.lmChatOpenAi)`

| Parameter      | Value          |
| -------------- | -------------- |
| **Model**      | gpt-5-nano     |
| **Credential** | OpenAI account |

#### Subnode 2.2: Structured Output Parser

**Type:** `Structured Output Parser (@n8n/n8n-nodes-langchain.outputParserStructured)`

**Purpose:** Parse classification result into structured format.

| Parameter          | Value                     |
| ------------------ | ------------------------- |
| **Schema Example** | `{ "prio": "Automated" }` |

---

#### Node 3: If

**Type:** `If (n8n-nodes-base.if)`

**Purpose:** Branch the workflow depending on whether the classification result equals “Automated.”

| Parameter     | Value                                  |
| ------------- | -------------------------------------- |
| **Condition** | `$json.output.prio` equals `Automated` |

---

#### Node 4: Chatbot

**Type:** `Agent (@n8n/n8n-nodes-langchain.agent)`

**Purpose:** Quick Q&A for issues that can be handled automatically.

| Parameter         | Value                                                       |
| ----------------- | ----------------------------------------------------------- |
| **Prompt Type**   | define                                                      |
| **Text Input**    | `{{ $('When chat message received').item.json.chatInput }}` |
| **System Prompt** |                                                             |

```markdown
### **Role**

You are an **IT Support Chatbot** for a corporate helpdesk.
Your role is to provide **first-level technical support** to employees who use **Windows** and **Microsoft Office applications** (Word, Excel, Outlook, PowerPoint, Teams, etc.).

### **Your Objectives:**
1. Ask the user about their name
2. Understand the user’s problem clearly.
3. Once the user provided their name and problem , provide **immediate troubleshooting steps** or quick answers when possible. Do not provide help until you know the name and problem.

### **Response Guidelines:**
* Use short paragraphs or numbered steps.
* When suggesting commands or settings, write them exactly as they appear in Windows or Office.
* Ask follow-up questions as needed.
* If you don't know the answer, say: "Sorry, I can't help with that."

### **Example Response Style:**

**User:** “I can’t open Excel files.”
**Chatbot:**

> Let’s try a quick fix:
>
> 1. Close all Excel windows.
> 2. Press **Windows + R**, type `excel /safe`, and press **Enter** to open Excel in Safe Mode.
> 3. Try opening the file again.
>
> Did that work? If not, we can check add-ins next or escalate to IT support.  
```

#### Subnode 4.1: OpenAI Chat Model

| Parameter      | Value          |
| -------------- | -------------- |
| **Model**      | gpt-5-nano     |
| **Credential** | OpenAI account |

#### Subnode 4.2: Simple Memory

**Type:** `Memory Buffer Window (@n8n/n8n-nodes-langchain.memoryBufferWindow)`

**Purpose:** Maintain short-term conversational context that is shared by both agents.

| Parameter                 | Value |
| ------------------------- | ----- |
| **Context Window Length** | 10    |

---

#### Node 5: Ticket Writer

**Type:** `Agent (@n8n/n8n-nodes-langchain.agent)`

**Purpose:** Create a support ticket and response when the issue must be routed to IT.

| Parameter         | Value                                                       |
| ----------------- | ----------------------------------------------------------- |
| **Prompt Type**   | define                                                      |
| **Text Input**    | `{{ $('When chat message received').item.json.chatInput }}` |
| **System Prompt** |                                                             |

```markdown
Create a ticket given the message history and a response to the user (letting them now that their request has been forwarded to IT and they'll be in touch soon) using the following fields:

- Name: <name of the user>
- Issue description: <summary of the issue, 1-2 sentences max.>
- Ticket ID: {{ $json.ID }}
- Response: <personal response, 1 sentence.>
```

#### Subnode 5.1: OpenAI Chat Model

**Type:** `OpenAI Chat Model (@n8n/n8n-nodes-langchain.lmChatOpenAi)`

| Parameter            | Value      |
| -------------------- | ---------- |
| **Model**            | gpt-5-nano |
| **Reasoning Effort** | low        |

#### Subnode 5.2: Structured Output Parser

**Type:** `Structured Output Parser (@n8n/n8n-nodes-langchain.outputParserStructured)`

| Parameter          | Value |
| ------------------ | ----- |
| **Schema Example** |       |

```json
{
  "Name": "Tobias",
  "Description": "My laptop fell down",
  "Response": "Sorry to hear that! IT will be in touch shortly!"
}
```

#### Subnode 5.3: Simple Memory

**Type:** `Memory Buffer Window (@n8n/n8n-nodes-langchain.memoryBufferWindow)`

| Parameter                 | Value |
| ------------------------- | ----- |
| **Context Window Length** | 10    |

---

#### Node 6: Edit Fields

**Type:** `Set (n8n-nodes-base.set)`

**Purpose:** Generate a unique ticket ID.

| Parameter      | Value                                       |
| -------------- | ------------------------------------------- |
| **Field Name** | ID                                          |
| **Expression** | `={{ $now.ts.toString(36).toUpperCase() }}` |
| **Type**       | string                                      |

---

#### Node 7: Urgency

**Type:** `LLM Chain (@n8n/n8n-nodes-langchain.chainLlm)`

**Purpose:** Classify ticket urgency based on description.

| Parameter         | Value                                                             |
| ----------------- | ----------------------------------------------------------------- |
| **Prompt Type**   | define                                                            |
| **Text Input**    | `Issue:\n\n{{ $('Ticket Writer').item.json.output.Description }}` |
| **System Prompt** |                                                                   |

```markdown
**Role**

You are a text classification system for an IT support helpdesk.
Your task is to read an incoming support ticket and classify it as either **"Urgent"** or **"Not urgent"** based on the content.

### Classification Rules:

* **Urgent** if the ticket describes:

  * Complete system or network outages.
  * Inability to perform critical work tasks.
  * Security breaches or data loss.
  * Hardware failure affecting essential operations.
  * Issues impacting multiple users or key business functions.
  * Time-sensitive problems needing immediate attention.

* **Not urgent** if the ticket describes:

  * Minor bugs or usability issues.
  * Requests for information, training, or feature changes.
  * Scheduled maintenance or non-critical updates.
  * Problems with available workarounds.
  * Single-user issues not blocking essential tasks.

### Output Format:

Respond **only** with one of the following labels:

* `Urgent`
* `Not urgent`

```

#### Subnode 7.1: OpenAI Chat Model

| Parameter      | Value          |
| -------------- | -------------- |
| **Model**      | gpt-5-nano     |
| **Credential** | OpenAI account |

#### Subnode 7.2: Structured Output Parser

**Type:** `Structured Output Parser (@n8n/n8n-nodes-langchain.outputParserStructured)`

| Parameter          | Value                  |
| ------------------ | ---------------------- |
| **Schema Example** | `{ "prio": "Urgent" }` |

---

#### Node 8: Create a file

**Type:** `GitHub (n8n-nodes-base.github)`

**Purpose:** Save a ticket file to the GitHub repository.

| Parameter          | Value                                                                                                                                                                                   |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Authentication** | oAuth2                                                                                                                                                                                  |
| **Owner**          | tobiaszwingmann                                                                                                                                                                         |
| **Repository**     | demo-building-ai-workflows-and-agents-with-n8n                                                                                                                                          |
| **File Path**      | `project 3/tickets/{{ $('Edit Fields').item.json.ID }}.txt`                                                                                                                             |
| **File Content**   | `Prio: {{ $json.output.prio }}\n\nName: {{ $('Ticket Writer').item.json.output.Name }}\n\nSubmitted: {{ $now }}\n\nDescription:\n{{ $('Ticket Writer').item.json.output.Description }}` |
| **Commit Message** | new ticket                                                                                                                                                                              |

---

#### Node 9: Edit Fields1

**Type:** `Set (n8n-nodes-base.set)`

**Purpose:** Compose a final chat message including the ticket ID and closing text.

| Parameter        | Value                                                                                                                                                                  |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Output Field** | output                                                                                                                                                                 |
| **Value**        | `={{ $('Ticket Writer').item.json.output.Response }}\n\nYour Ticket ID is: {{ $('Edit Fields').item.json.ID }}.\n\nThis chat will now be closed.\n\nHave a great day!` |

---

## Sample Queries

Use the following to test this workflow interactively:

* “I can’t log in”
* “My laptop fell down and is broken”
* “Talk to a human”
