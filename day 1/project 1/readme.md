# Build your first basic automation

## Workflow Overview
**Purpose:** Collect IT support requests, generate a unique ticket ID, and create a corresponding text file in GitHub.

**Trigger:** Form submission (`Form Trigger`)

**Nodes:**

1. On form submission
2. Edit Fields
3. Create a file

<img width="683" height="269" alt="image" src="https://github.com/user-attachments/assets/2e33edfc-a599-4eae-98be-e4be59e3b763" />


---

## Node 1: On form submission
**Type:** `Form Trigger`

**Purpose:** Collect user input through an IT Service Request form.

| Parameter            | Value                                                              					 |
| -------------------- | ----------------------------------------------------------------------------------------|
| **Form Title**       | IT Service Request                                                 					 |
| **Form Description** | Submit your issue here                                             					 |
| **Form Fields**      | **Add Form Element** <br>- Issue description (textarea, required)<br>- Your Name (text input, required) |
| **Add Option**       | **Form Response** <br>"IT support will be in touch shortly!"                            		 |

---

## Node 2: Edit Fields

**Type:** `Edit Fields (Set)`

**Purpose:** Generate a unique ticket ID from the form submission timestamp.

| Parameter                            | Value                                                                 |
| ------------------------------------ | --------------------------------------------------------------------- |
| **Field Name**                       | ID                                                                    |
| **Expression**                       | `{{ $json.submittedAt.toDateTime().ts.toString(36).toUpperCase() }}`  |
| **Type**                             | string                                                                |
| **Include Other Input Fields**       | True                                                                  |


**Rename this Node:** `Set ID`

---

## Node 3: Create a file

**Type:** `GitHub`

**Purpose:** Save a new ticket file to a GitHub repository.

| Parameter | Value |
|---|---|
| **Authentication** | `oAuth2` |
| **Owner** | `your GitHub username` |
| **Repository** | `your GitHub repo` |
| **File Path** | `day 1/tickets/{{ $('Set ID').item.json.ID }}.txt` |
| **Commit Message** | `new ticket` |

**File Content**
```
Name: {{ $('Set ID').item.json['Your Name'] }}

Submitted:{{ $('Set ID').item.json.submittedAt }}

Issue: {{ $('Set ID').item.json['Issue description'] }}
```
