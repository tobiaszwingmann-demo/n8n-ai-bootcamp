# P8 – AI Agent with MCP

This project contains **two workflows** that together demonstrate how to expose n8n functionality as **MCP tools** and use them from an AI agent.

* MCP Server → exposes ticket operations as tools
* MCP Client → AI chatbot that uses those tools

## Part 1: Creating the MCP Server
**Purpose:** Serve a collection of tools via MCP for easy tool access, maintenance, and shareability.

### 1. Create New Workflow
- n8n home --> New workflow
- Name the workflow `P8 - MCP Server`

### 2. Add `MCP Server Trigger` node
- **Authentication**: None

### 3. Add custom tools
- Open a new tab and navigate to your previous workflow **P7 - Advanced AI Agent**
- **Select** and **Copy** the following tools:
  - **Update Ticket**
  - **Get Ticket**
  - **Create Ticket**
 (Right click --> **Copy 3 Nodes**
- Back to your `P8 - MCP Server` workflow
- Paste the tools
- **Connect the tools** to the MCP Server Node

### 4. Publish your MCP Server
- Hit Publish
- Open the **MCP Server** Node and **copy the production URL**

## Part 2: Creating the MCP Client
**Purpose:** Access our MCP server from our AI Agent.

### 1. Duplicate or Copy Workflow
- n8n home --> `P7 - Advanced Agent` --> Duplicate
- Name: `P8 - MCP Client`

### 2. Open the workflow `P8 - MCP Client`
- **Delete the following tools:**
  - **Update Ticket**
  - **Get Ticket**
  - **Create Ticket**

### 3. Add the MCP Client
- Click the `+` icon under Agent Tool
- Select **MCP Client Tool**
Parameters:
- **Endpoint:** The **Production URL** from `P8 - MCP Server`
- **Tools to Include**: `Selected`
- Confirm the tools are visible. If not, double check you published your server and pasted the correct URL.
- See the tools? **Select all**

### 4. Try the connection
- Ask your AI Agent to retrieve a ticket ID that is available on Github, e.g. `MLFNSMXT`
- Works? Your MCP Server is up and running
To-Do:
- Try creating a new Tickets or updating existing tickets

