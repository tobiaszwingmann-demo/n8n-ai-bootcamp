# P2 – Build a simple AI-powered text classifier

**Note:** Continue using the workflow from project 1 and then modify it as described below.

## Workflow Overview

**Purpose:** Collect IT support requests, classify them as *Urgent* or *Not urgent* using an LLM, and create a ticket file in GitHub with the classification result.

**Trigger:** Form submission (`Form Trigger`)

**Nodes:**

1. On form submission
2. Edit Fields
3. Basic LLM Chain
   3.1 Chat Model
4. Create a file

<img width="683" height="269" alt="image" src="https://github.com/user-attachments/assets/558ecf1f-5cc2-4da7-8935-e2979b857578" />

                   
---

#### Node 3: Basic LLM Chain

**Type:** `LLM Chain`

**Purpose:** Classify the IT issue as *Urgent* or *Not urgent* based on predefined rules.

| Parameter      | Value                                                                 |
| -------------- | --------------------------------------------------------------------- |
| **Prompt Source** | Define below                                                       |
| **Require Output Format** | Toggle On |

**Prompt (User Message)**
````markdown
Issue:

```
{{ $json['Issue description'] }}
```
````

**System Prompt**
```markdown
### **Role**

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

#### Subnode 3.1: Chat Model

- **Type:** Pick any model you like
- For our demo, you can use the **OpenAI free credits** that come wiht n8n
- For advanced use, create an [API Key](https://platform.openai.com/api-keys) and store it as a new credential in n8n.

#### Subnode 3.2: Structured Output Parser

-  **Purpose:** Parse the model’s output into a structured JSON format.

| Parameter          | Value                  |
| ------------------ | ---------------------- |
| **Schema Example** | `{ "prio": "Urgent" }` |

---

#### Node 4: Create a file

**Type:** `GitHub`

**Purpose:** Create a ticket file in GitHub including classification, submitter info, and issue details.

| Parameter          | Value                                                                                                                                                                                                                                            |
| ------------------ | -------------------------------------------------------- |
| **Authentication** | oAuth2                                                                                                                                                                                                                                           |
| **Resource**          | File                                                                                                                                                                                                                                          |
| **Operation**     | Create                                                    |
| **File Path**      | `day 1/tickets/{{ $('Set ID').item.json.ID }}.txt`  |
| **Commit Message** | new ticket                                                                                                                                                                                                                                       |

**File Content**
```markdown

Prio: {{ $json.output.prio }}

Name: {{ $('Set ID').item.json['Your Name'] }}

Submitted:{{ $('Set ID').item.json.submittedAt }}

Issue: {{ $('Set ID').item.json['Issue description'] }}
```

