# 🧠 Model Context Protocol (MCP)

> A standardized protocol for AI hosts to connect with external tools, data sources, and services — securely, scalably, and without tight coupling.

---

## 📐 Architecture

MCP follows a **Hub-and-Spoke** model. A single Host process contains one or more Clients, each maintaining a dedicated **1:1 connection** to a Server.

```
┌──────────────────────────┐
│      🖥️  Host Process     │
│                          │
│  ┌──────────────────┐    │
│  │    Client #1     │────────────▶  Server A (Database)
│  └──────────────────┘    │
│  ┌──────────────────┐    │
│  │    Client #2     │────────────▶  Server B (File System)
│  └──────────────────┘    │
│  ┌──────────────────┐    │
│  │    Client #3     │────────────▶  Server C (Web Search)
│  └──────────────────┘    │
└──────────────────────────┘
```

### Why This Design?

| Benefit                   | Description                                                               |
| ------------------------- | ------------------------------------------------------------------------- |
| 🔒 **Decoupled & Safe**   | Each server runs in isolation — a compromised server cannot affect others |
| ⚡ **Parallel Execution** | Multiple clients can call multiple servers simultaneously                 |
| 📈 **Scalable**           | Add new capabilities by connecting new servers, no host changes needed    |

---

## 🧩 MCP Primitives

Every capability in MCP is expressed through one of **three fundamental abstractions**:

```
┌─────────────┐   ┌─────────────────┐   ┌──────────────────┐
│   ⚙️ Tools   │   │  📦 Resources   │   │   💬 Prompts     │
│             │   │                 │   │                  │
│  Actions    │   │  Structured     │   │  Predefined      │
│  the AI     │   │  data the AI    │   │  templates that  │
│  can invoke │   │  can read       │   │  shape behavior  │
└─────────────┘   └─────────────────┘   └──────────────────┘
```

| Primitive     | Role                                          | Analogy               |
| ------------- | --------------------------------------------- | --------------------- |
| **Tools**     | Actions the AI asks the server to perform     | Function calls        |
| **Resources** | Structured data sources the AI can read       | A readable filesystem |
| **Prompts**   | Predefined prompt templates the server offers | Workflow starters     |

---

## 🔧 Standard Operations

### ⚙️ Tools

| Operation    | Direction       | Description                                                                     |
| ------------ | --------------- | ------------------------------------------------------------------------------- |
| `tools/list` | Client → Server | "What tools do you expose?" Returns all available tools and their input schemas |
| `tools/call` | Client → Server | "Execute this tool with these arguments" — the primary action mechanism         |

### 📦 Resources

| Operation               | Direction       | Description                                                  |
| ----------------------- | --------------- | ------------------------------------------------------------ |
| `resources/list`        | Client → Server | "What resources are available?" Returns name, URI, MIME type |
| `resources/subscribe`   | Client → Server | "Notify me when this resource changes"                       |
| `resources/unsubscribe` | Client → Server | "Stop sending updates for this resource"                     |

### 💬 Prompts

| Operation      | Direction       | Description                                                      |
| -------------- | --------------- | ---------------------------------------------------------------- |
| `prompts/list` | Client → Server | "What prompt templates do you offer?"                            |
| `prompts/get`  | Client → Server | "Give me this specific template" with optional dynamic arguments |

---

## 📦 Data Layer — JSON-RPC 2.0

All MCP messages are encoded as **JSON-RPC 2.0** — a lightweight remote procedure call protocol built on JSON.

> **What is RPC?** A Remote Procedure Call lets a program invoke a function on a remote computer _as if it were local_, hiding all the networking complexity behind a clean interface. Instead of writing `add(2, 3)` locally, you send a structured message to the server asking it to run `add` for you.

### Message Format

**Request:**

```json
{
  "jsonrpc": "2.0",
  "method": "tools/call",
  "params": {
    "name": "add",
    "args": [2, 3]
  },
  "id": 1
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "result": 5,
  "id": 1
}
```

### Why JSON-RPC for MCP?

| Feature                   | Why It Matters                                                     |
| ------------------------- | ------------------------------------------------------------------ |
| 🪶 **Lightweight**        | Tiny human-readable JSON payloads vs heavy XML alternatives        |
| ↔️ **Bidirectional**      | Both client and server can initiate messages                       |
| 🚂 **Transport-Agnostic** | Same format works over STDIO, HTTP, WebSockets, or any byte stream |
| 📦 **Batching**           | Multiple RPC calls in one payload — reduces round-trip overhead    |
| 🔔 **Notifications**      | Fire-and-forget messages with no response required                 |

---

## 🚀 Transport Layer

The Transport Layer is the **physical mechanism** that carries JSON-RPC messages between the Client and Server. MCP supports two transport types:

```
         ┌──────────────────────────────────────────────────┐
         │              Transport Layer                     │
         │                                                  │
         │   ┌────────────────┐    ┌─────────────────────┐  │
         │   │  Local Server  │    │   Remote Server     │  │
         │   │   via STDIO    │    │  via HTTP + SSE     │  │
         │   └────────────────┘    └─────────────────────┘  │
         └──────────────────────────────────────────────────┘
```

---

### 🖥️ Local Server — STDIO Transport

The host launches the server as a **subprocess** on the same machine. Communication happens via Standard Input/Output (stdin/stdout).

```
Host (Client)           Server Subprocess
      │                        │
      │──── JSON-RPC ──────▶ STDIN
      │                        │  (processes request)
   STDOUT ◀──── response ──────│
```

**Benefits:**

|               |                                                                  |
| ------------- | ---------------------------------------------------------------- |
| ✅ **Fast**   | Data passed directly between processes — zero network overhead   |
| ✅ **Secure** | No open network port; communication is purely local              |
| ✅ **Simple** | Every language supports stdin/stdout — no extra libraries needed |

---

### 🌐 Remote Server — HTTP + SSE Transport

The server lives on a **different machine** (cloud, network). The host sends requests via HTTP POST and receives streaming responses via Server-Sent Events.

```
Host (Client)                    Remote Server
      │                               │
      │──── HTTP POST (JSON-RPC) ──▶  │
      │                               │  (processes...)
      │  ◀── SSE stream (chunks) ─────│
      │  ◀── SSE stream (chunks) ─────│
      │  ◀── SSE stream (done)  ──────│
```

**Benefits:**

|                           |                                                            |
| ------------------------- | ---------------------------------------------------------- |
| ✅ **Works Anywhere**     | Reach servers running anywhere on the internet             |
| ✅ **Standard Auth**      | Supports API keys, OAuth, and other HTTP auth methods      |
| ✅ **Streaming**          | Receive incremental results via SSE as they're ready       |
| ✅ **Long-Running Tasks** | Perfect for tasks that take time — no timeout hacks needed |

---

### 📡 What is SSE (Server-Sent Events)?

SSE is an HTTP extension that keeps a connection **open after the initial response**, letting the server push multiple messages over a single connection.

- 📥 **One-directional** — server → client only (unlike WebSockets)
- 🔄 **Streaming** — chunks arrive as soon as they're ready, no polling
- 🏁 **Ideal for** — long-running tool calls, progress updates, log streaming

---

## 🗺️ Summary

```
┌─────────────────────────────────────────────────┐
│                  MCP Stack                      │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │           Application Layer             │   │
│  │    Tools │ Resources │ Prompts          │   │
│  └─────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────┐   │
│  │             Data Layer                  │   │
│  │          JSON-RPC 2.0                   │   │
│  └─────────────────────────────────────────┘   │
│  ┌─────────────────────────────────────────┐   │
│  │           Transport Layer               │   │
│  │     STDIO (local) │ HTTP+SSE (remote)   │   │
│  └─────────────────────────────────────────┘   │
└─────────────────────────────────────────────────┘
```

---

_MCP — Model Context Protocol · Technical Reference_
