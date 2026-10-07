# P5 – Simple Support System

**Important:** **Copy your workflow form P3** and **paste it into the workflow with the Q&A Chatbot (P4).** 

---

## Workflow Overview

**Purpose**
This workflow demonstrates a simple version of an internal IT support chat system that is able to solve simple questions automatically while being able to escalate issues when necessary.

Incoming chat messages are automatically classified and either:

* handled directly by an AI chatbot, or
* escalated into a structured IT support ticket for human follow-up.

<img width="683" height="269" alt="image" src="https://github.com/user-attachments/assets/7600dcfc-fb61-4c91-9db3-8e7355d3db25" />

---

### Q&A Chatbot Agent

**Add to the System Prompt**

Add the following part to the system prompt of your Q&A Chatbot (for example after the Response guidelines and examples)

```
### **Escalation Rule:**
If the request includes **Keywords or topics** such as:
- *hardware*, *broken*, *damaged*, *replacement*, *repair*
- *network down*, *server*, *outage*, *system crash*
- *admin rights*, *installation*, *permissions*, *access denied*
- *security*, *breach*, *virus*, *phishing*, *data loss*
- *urgent*, *critical*, *can’t work*, *system not starting*
- *new equipment*, *device setup*, *hardware request*
- *custom software* like NovaCRM
- *user demands human assistance*
DO NOT ANSWER the question. 

Instead, respond with "CODE: ESCALATION"
```

### If Node

`{{ $json.output }}` contains `CODE: ESCALATION`

### AI Agent

**Prompt:**

```
Summarize the user issue and return JSON with the user name and a short issue description.
```

- Require Specific Output Format: `True`
- Memory: *Connect to existing memory*
- Model: *Connect to existing chat model*

**Structured Output Parser – Generate from Example**
```
{
  "Issue description": "My laptop broke",
  "Name": "Tobias"
}
```

- Rename to: **Summary Agent**

### Set ID Node (Update)

| Name | Type | Value |
|---|---|---|
| `ID` | String | `{{ $now.toDateTime().ts.toString(36).toUpperCase() }}` |
| `submittedAt` | String | `{{$now}}` |
| `Issue description` | String | `{{ $json.output['Issue description'] }}` |
| `Your Name` | String | `{{ $json.output['Name'] }}` |

- **Include other input fields:** `False`

### Edit Fields Node (New)

- `output`
- String
```
IT support will be in touch soon!

Your Ticket ID is {{ $('Set ID').item.json.ID }}
```
