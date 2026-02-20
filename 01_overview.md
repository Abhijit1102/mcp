# MCP — Model Context Protocol

> **Protocol Documentation · v1.0**

_The universal standard for connecting AI assistants to the tools and data they need — securely, simply, and at scale._

---

|   📅 Nov '22   | ⚡ 5 Days | 🚀 2 Months |  🔌 Nov '24  |
| :------------: | :-------: | :---------: | :----------: |
| ChatGPT Launch | 1M Users  | 100M Users  | MCP Released |

---

## Table of Contents

- [01 — The Journey: How We Got Here](#01--the-journey-how-we-got-here)
- [02 — The Problem: The Fragmentation Crisis](#02--the-problem-the-fragmentation-crisis)
- [03 — Foundation: What is Context?](#03--foundation-what-is-context)
- [04 — Evolution: From Function Calling to MCP](#04--evolution-from-function-calling-to-mcp)
- [05 — Architecture: How MCP Works](#05--architecture-how-mcp-works)
- [06 — Division of Labor](#06--division-of-labor)
- [07 — MCP vs Function Calling](#07--mcp-vs-function-calling)
- [08 — Why MCP Wins](#08--why-mcp-wins)
- [09 — The MCP Ecosystem Flywheel](#09--the-mcp-ecosystem-flywheel)

---

## 01 — The Journey: How We Got Here

The story of MCP begins with the most explosive software launch in history — and the fragmentation that followed.

---

### 🟣 Origin — Nov 2022 · ChatGPT Changes Everything

OpenAI's ChatGPT launches and reaches **1 million users in just 5 days**. 100 million in 2 months. Not a software launch — a civilizational moment.

---

### 🟢 Wave 1 — Pure Wonder · The World Discovers AI

Millions of people chat with an AI for the first time. The experience is magical and surprising. Everyone has a story to tell.

---

### 🔴 Wave 2 — Professional Adoption · AI Enters the Workplace

Professionals integrate AI into daily workflows. Productivity climbs but silos emerge. AI in Notion can't talk to AI in Slack. VS Code knows nothing about MS Teams.

---

### 🟡 Wave 3 — The API Revolution · Developers Build on Top

OpenAI's API opens the floodgates. Function Calling arrives mid-2023, letting LLMs invoke external tools. Thousands of custom integrations are built — each one bespoke.

---

### 🟠 Wave 4 — Embedded AI Tools · AI Inside Your Workflow

Products like Cursor and Perplexity embed AI directly inside developer tools. AI becomes ambient — everywhere, but still deeply fragmented.

---

## 02 — The Problem: The Fragmentation Crisis

As AI proliferated, a deep structural problem emerged: every AI lived on its own island, blind to the others. Users wanted one unified AI partner. Instead they got five.

> _"Users were juggling multiple AI assistants. AI in Notion couldn't talk to AI in Slack. VS Code's coding assistant knew nothing about discussions in MS Teams."_
>
> — The Reality of Multi-AI Workflows, 2023–2024

### The Four Failure Modes

**🏝 Isolated Islands**
Each AI tool operated in isolation with its own context, memory, and permissions — unable to share state with peers.

**🔄 Context Switching Tax**
Users manually copy-pasted between tools, reconstructing context every time they switched assistants — burning time and losing nuance.

**🔧 Redundant Integrations**
Every AI company built its own GitHub connector, Slack integration, Drive bridge — duplicating enormous effort across the industry.

**🔒 Security Chaos**
Auth tokens scattered across dozens of integrations with inconsistent security models and no unified access policy.

---

## 03 — Foundation: What is Context?

Context is everything an AI can "see" when generating a response — all the information an LLM uses to produce its output. More context = smarter, more relevant answers.

| Type                        | Description                                                                                                    |
| --------------------------- | -------------------------------------------------------------------------------------------------------------- |
| 💬 **Conversation History** | Every message in a chat session. The AI sees everything said before.                                           |
| 📄 **External Documents**   | Files, PDFs, spreadsheets, and web pages injected so the AI can reason over real-world data.                   |
| 🛠 **Tool Results**         | When an AI calls a function or API, the returned data becomes context — enabling multi-step reasoning.         |
| 🧠 **System Instructions**  | A system prompt that configures the AI's behavior and persona — invisible to users but shaping every response. |

---

## 04 — Evolution: From Function Calling to MCP

Function Calling was the prototype. It connected LLMs to external tools, unlocking integrations for Salesforce, Slack, Drive, GitHub, and more. But it created an **N×M maintenance nightmare**.

### ⚠️ The Old World — N×M Integration Chaos

```
Jira → ChatGPT → GitHub → ChatGPT → MySQL → ChatGPT → Drive → ChatGPT → Slack
```

Every hop is a custom integration. Every AI client must rebuild the same connectors.
**Maintenance burden = N clients × M tools.**

---

### ✅ The MCP World — Build Once, Connect Everywhere

```
Any AI Client  ⟷  MCP Protocol  ⟷  GitHub Server
                                ⟷  Slack Server
                                ⟷  MySQL Server
                                ⟷  Drive Server
```

Every server speaks the same language. Any new MCP client instantly works with every existing server.

---

## 05 — Architecture: How MCP Works

MCP follows a clean **client-server architecture** over **JSON-RPC 2.0**. The AI model acts as a client; external services expose themselves as MCP servers. Communication is standardized, secure, and discoverable.

```
┌─────────────────┐        MCP Protocol        ┌──────────────────┐       Native API      ┌──────────────────┐
│   🤖 MCP Client │  ◀────────────────────▶   │  ⚙️  MCP Server  │  ◀─────────────────▶  │   🗄 Resources    │
│                 │                             │                  │                       │                  │
│  Claude         │                             │  Auth & Limits   │                       │  GitHub / GitLab │
│  ChatGPT        │                             │  Data Formatting │                       │  Google Drive    │
│  Cursor         │                             │  Error Handling  │                       │  Slack / Teams   │
│  Copilot        │                             │  Business Logic  │                       │  Databases / APIs│
└─────────────────┘                             └──────────────────┘                       └──────────────────┘
```

---

## 06 — Division of Labor

MCP's genius is in **where it places complexity**. The server absorbs all integration burden — the client stays clean and simple.

### ⚙️ MCP Server Handles

- Authentication with GitHub / OAuth flows
- API rate limiting and retry logic
- Data format translation and normalization
- Error handling and graceful degradation
- Service-specific business logic
- Schema validation and type safety
- Pagination and streaming responses

### 🤖 MCP Client Just Needs To

- Connect to an MCP server
- Discover available tools
- Call tools with parameters
- Use the returned context

_That's it. Nothing more._

---

## 07 — MCP vs Function Calling

Function Calling was the prototype; MCP is the standard.

| Dimension            | ⚠️ Function / Tool Calling                 | ✅ MCP                              |
| -------------------- | ------------------------------------------ | ----------------------------------- |
| **Scope**            | Single AI provider, custom per integration | Universal — any client, any server  |
| **Integration Code** | Written separately for every AI client     | Written once on the server side     |
| **Maintenance**      | ✗ N clients × M tools = crushing burden    | ✓ Only the MCP server is maintained |
| **Security Model**   | Fragmented — varies per integration        | ✓ Unified auth and permission model |
| **Tool Discovery**   | Hardcoded function schemas                 | ✓ Dynamic discovery at runtime      |
| **Transport**        | Embedded in the API call                   | ✓ stdio, HTTP/SSE, WebSocket        |
| **Ecosystem**        | Siloed per LLM provider                    | ✓ Shared ecosystem, open standard   |

---

## 08 — Why MCP Wins

MCP solves the integration problem at the **protocol level** — not the product level. The benefits compound as the ecosystem grows.

**01 · Zero Duplication**
Integration logic lives once — on the server. Every AI client inherits it for free. No more rewriting the same connector per product.

**02 · Unified Security**
Auth, permissions, and rate limiting handled once on the server. Audit trails and access controls are centralized and consistent.

**03 · Lower Cost**
Teams no longer maintain N×M integrations. Build the MCP server once — engineering costs drop dramatically across the board.

**04 · Faster Iteration**
Update the server and all clients get improvements instantly. No coordinated releases across multiple AI platform integrations.

**05 · Rich Ecosystem**
A growing library of community MCP servers means you often don't need to build from scratch. Plug in and go.

**06 · Future-Proof**
New AI models that adopt MCP automatically gain access to your entire server catalog — no additional integration work ever.

---

## 09 — The MCP Ecosystem Flywheel

MCP creates a powerful **network-effect flywheel**. Each new participant makes the entire ecosystem more valuable for everyone else.

```
          ┌─────────────────────┐
          │   More AI Clients   │
          │  Claude, Cursor, Zed│
          └──────────┬──────────┘
                     │
         ┌───────────▼───────────┐
 User    │                       │  More
Adoption │     MCP Ecosystem     │  Servers
grows ──▶│                       │──▶ GitHub
demand   │                       │    Drive, DB…
         └───────────┬───────────┘
                     │
          ┌──────────▼──────────┐
          │     More Value      │
          │   for all services  │
          └─────────────────────┘
```

### The Ecosystem Today

**🏗 Growing Server Catalog**
Hundreds of community and official MCP servers already exist for GitHub, Slack, Google Drive, databases, Figma, Linear, Salesforce, and more.

**🤝 Industry-Wide Adoption**
Claude, Cursor, Zed, Replit, Codeium, and many others have adopted MCP as a native integration standard since its release.

**📖 Open Standard**
MCP is fully open-source, published by Anthropic. The spec, SDKs (Python, TypeScript), and reference servers are freely available for anyone.

---

## Resources

| Resource                | Link                                                                       |
| ----------------------- | -------------------------------------------------------------------------- |
| 📚 Official Spec & Docs | [modelcontextprotocol.io](https://modelcontextprotocol.io)                 |
| 💻 Open Source SDKs     | [github.com/modelcontextprotocol](https://github.com/modelcontextprotocol) |

---

_Model Context Protocol — Published by Anthropic, November 2024_
