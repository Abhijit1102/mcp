# 🔌 MCP Lifecycle — Model Context Protocol

> **MCP (Model Context Protocol)** defines a structured communication protocol between a **Host (Client)** and a **Server**. The lifecycle describes every step — from the first handshake to graceful shutdown — that governs a session.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Initialization Phase](#initialization-phase)
  - [Step 1 — Client Sends Initialize Request](#step-1--client-sends-initialize-request)
  - [Step 2 — Server Responds with Capabilities](#step-2--server-responds-with-capabilities)
  - [Step 3 — Client Sends Initialized Notification](#step-3--client-sends-initialized-notification)
- [Capability Negotiation](#capability-negotiation)
- [Operation Phase](#operation-phase)
  - [Capability Discovery](#capability-discovery)
  - [Tool Calling](#tool-calling)
- [Shutdown Phase](#shutdown-phase)
  - [Process-Based Shutdown](#process-based-shutdown)
  - [HTTP-Based Shutdown](#http-based-shutdown)
- [Special Cases & Utilities](#special-cases--utilities)
  - [Ping / Health Check](#ping--health-check)
  - [Error Handling](#error-handling)
  - [Timeouts & Cancellation](#timeouts--cancellation)
  - [Progress Notifications](#progress-notifications)

---

## Overview

The MCP lifecycle has three major phases:

```
┌─────────────────────────────────────────────────────┐
│                   MCP SESSION                        │
│                                                      │
│  ┌────────────────┐  ┌───────────┐  ┌────────────┐  │
│  │ Initialization │→ │ Operation │→ │  Shutdown  │  │
│  └────────────────┘  └───────────┘  └────────────┘  │
└─────────────────────────────────────────────────────┘
```

| Phase              | Purpose                                                         |
| ------------------ | --------------------------------------------------------------- |
| **Initialization** | Establish connection, negotiate protocol version & capabilities |
| **Operation**      | Exchange messages, call tools, subscribe to resources           |
| **Shutdown**       | Gracefully terminate the connection                             |

---

## Initialization Phase

> The initialization phase **MUST** be the very first interaction between the Client and Server. Think of it as the handshake — before any real work begins, both sides agree on _what they can do_ and _how they'll talk_.

### What happens during initialization?

- ✅ Protocol version compatibility is established
- ✅ Capabilities are exchanged and negotiated
- ✅ Both sides confirm they are ready to operate

---

### Step 1 — Client Sends Initialize Request

The client kicks off the session by declaring its protocol version, capabilities, and identity.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "protocolVersion": "2025-03-26",
    "capabilities": {
      "roots": {
        "listChanged": true
      },
      "sampling": {},
      "elicitation": {}
    },
    "clientInfo": {
      "name": "IDEPlugin",
      "version": "1.0.0"
    }
  }
}
```

**Field Breakdown:**

| Field             | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `protocolVersion` | The MCP protocol version the client wants to use             |
| `capabilities`    | Features this client supports (roots, sampling, elicitation) |
| `clientInfo`      | Human-readable name and version of the client application    |

---

### Step 2 — Server Responds with Capabilities

The server replies with its own supported capabilities and identity, completing the negotiation.

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "protocolVersion": "2025-03-26",
    "capabilities": {
      "tools": {
        "listChanged": true
      },
      "resources": {
        "listChanged": true,
        "subscribe": true
      },
      "prompts": {
        "listChanged": true
      },
      "logging": {}
    },
    "serverInfo": {
      "name": "FileSystemServer",
      "version": "2.5.1"
    },
    "instructions": "Server is ready to accept commands"
  }
}
```

**Field Breakdown:**

| Field             | Description                                                        |
| ----------------- | ------------------------------------------------------------------ |
| `protocolVersion` | Must match (or be compatible with) the client's version            |
| `capabilities`    | Features this server supports (tools, resources, prompts, logging) |
| `serverInfo`      | Human-readable name and version of the server                      |
| `instructions`    | Optional human-readable startup message                            |

---

### Step 3 — Client Sends Initialized Notification

Once the client receives the server's response, it sends a **notification** (not a request — no response expected) to signal it is ready for normal operations.

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/initialized"
}
```

> 🎉 **At this point, the Client and Server are fully connected and ready to work.**

---

### ⚠️ Important Rules During Initialization

| Party      | Restriction                                                                                              |
| ---------- | -------------------------------------------------------------------------------------------------------- |
| **Client** | SHOULD NOT send requests other than `ping` before the server has responded to `initialize`               |
| **Server** | SHOULD NOT send requests other than `ping` and `logging` before receiving the `initialized` notification |

---

## Capability Negotiation

Capabilities define _which protocol features_ are available for the session. Each side advertises what it supports — only mutually supported features can be used.

### Client Capabilities

| Capability    | Description                                                             |
| ------------- | ----------------------------------------------------------------------- |
| `roots`       | Client can expose filesystem roots to the server                        |
| `sampling`    | Client supports LLM sampling requests from the server                   |
| `elicitation` | Client can prompt the user for additional input on behalf of the server |

### Server Capabilities

| Capability  | Description                                     |
| ----------- | ----------------------------------------------- |
| `tools`     | Server exposes callable tools                   |
| `resources` | Server exposes readable resources (files, data) |
| `prompts`   | Server exposes prompt templates                 |
| `logging`   | Server can send log messages to the client      |

### Sub-Capability Flags

| Flag          | Applies To                       | Meaning                                               |
| ------------- | -------------------------------- | ----------------------------------------------------- |
| `listChanged` | tools, resources, prompts, roots | Server/Client will notify when the list changes       |
| `subscribe`   | resources                        | Client can subscribe to resource change notifications |

---

## Operation Phase

During the operation phase, the client and server freely exchange messages — **within the boundaries of what was negotiated**.

### Rules

- ✅ Respect the negotiated protocol version
- ✅ Only use capabilities that were successfully negotiated
- ❌ Do not call methods for capabilities that were not agreed upon

---

### Capability Discovery

After initialization, the client typically asks the server what tools/resources/prompts are available.

**Client requests tool list:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "tools/list"
}
```

**Server responds:**

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": {
    "tools": [
      {
        "name": "listRepos",
        "description": "List repositories for a user or organization"
      },
      {
        "name": "getFile",
        "description": "Read a file from a repository by path and ref"
      },
      {
        "name": "searchCode",
        "description": "Search code across repositories"
      },
      {
        "name": "createIssue",
        "description": "Open a new GitHub issue"
      },
      {
        "name": "listPRs",
        "description": "List pull requests for a repository"
      }
    ]
  }
}
```

---

### Tool Calling

Once the client knows what tools exist, it can invoke them.

**Client calls a tool:**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "getFile",
    "arguments": {
      "owner": "campusx-official",
      "repo": "mcp-examples",
      "path": "README.md",
      "ref": "main"
    }
  }
}
```

**Server responds with result:**

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "content": "# MCP Examples\nThis repo contains MCP examples...",
    "encoding": "utf-8",
    "sha": "f3c0..."
  }
}
```

---

## Shutdown Phase

One side — **typically the client** — initiates shutdown. MCP does not define a special JSON-RPC shutdown message; instead, the **transport layer** signals termination.

### Process-Based Shutdown

When the server runs as a child process (e.g., stdio transport):

**Client-initiated (SHOULD):**

```
1. Close the input stream to the child process (server)
2. Wait for the server to exit gracefully
3. Send SIGTERM if the server does not exit in time
4. Send SIGKILL if the server is still unresponsive
```

**Server-initiated:**

```
1. Close the output stream to the client
2. Exit the process
```

---

### HTTP-Based Shutdown

When using HTTP/SSE transport:

**Client-initiated (most common):**

- The client closes all HTTP connections it opened to the server.

**Server-initiated:**

- The server closes the connection from its side.
- The client must detect the dropped connection and handle it (e.g., attempt reconnection if appropriate).

---

## Special Cases & Utilities

### Ping / Health Check

`ping` is a lightweight request/response mechanism to verify the other side is still alive and the connection is responsive. It can be sent by **either side** at **any time** — even before initialization completes.

**Request:**

```json
{
  "jsonrpc": "2.0",
  "id": 42,
  "method": "ping"
}
```

**Response:**

```json
{
  "jsonrpc": "2.0",
  "id": 42,
  "result": {}
}
```

**When to use ping:**

- Before full `initialize`, to check if the other side is reachable
- During long idle periods, to detect dropped connections
- As a lightweight keepalive mechanism

---

### Error Handling

MCP inherits **JSON-RPC 2.0's standard error object format**. When something goes wrong, the responding side returns an error object instead of a result.

#### Common Causes of Errors

- Unsupported or mismatched protocol version
- Calling a method for a capability that was not negotiated
- Invalid or missing arguments to a tool
- Internal server failure while processing a request
- Request timeout / client cancellation
- Malformed JSON-RPC message

#### Error Object Structure

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "error": {
    "code": -32601,
    "message": "Method not found",
    "data": {
      "extra": "Optional structured debugging info"
    }
  }
}
```

| Field     | Type                | Description                             |
| --------- | ------------------- | --------------------------------------- |
| `code`    | integer             | Categorizes the error (see table below) |
| `message` | string              | Short, human-readable explanation       |
| `data`    | object _(optional)_ | Extra structured context for debugging  |

#### Standard Error Codes

| Code               | Name                  | Meaning                                           |
| ------------------ | --------------------- | ------------------------------------------------- |
| `-32700`           | Parse Error           | Invalid JSON was received                         |
| `-32600`           | Invalid Request       | JSON-RPC structure is invalid                     |
| `-32601`           | Method Not Found      | The method does not exist or is not available     |
| `-32602`           | Invalid Params        | Invalid method parameters                         |
| `-32603`           | Internal Error        | Internal JSON-RPC error                           |
| `-32000` and below | Server-Defined Errors | Application-specific errors defined by the server |

---

### Timeouts & Cancellation

Timeouts protect against requests that hang forever — keeping resources free and users informed.

**How it works:**

1. The client sets a per-request timeout (e.g., 30 seconds) via the SDK.
2. If the deadline passes with no response, the client triggers a cancellation.
3. A `notifications/cancelled` message is sent to the server.
4. The server SHOULD stop processing and release resources.

**Initial tool call with a progress token (enables cancellation tracking):**

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "tools/call",
  "params": {
    "name": "searchCode",
    "arguments": { "query": "MCP" },
    "_meta": { "progressToken": "tok-7" }
  }
}
```

**Cancellation notification sent by the client on timeout:**

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/cancelled",
  "params": {
    "requestId": 7,
    "reason": "Timeout exceeded (30s)"
  }
}
```

> ℹ️ `_meta.progressToken` links the cancellation notification back to the original request — the server uses this token to know which operation to stop.

---

### Progress Notifications

For long-running operations, the server can stream progress updates back to the client — so users aren't left staring at a spinner with no feedback.

**How it works:**

1. Client includes a `progressToken` in the request's `_meta` field.
2. While working, the server sends one or more `notifications/progress` messages.
3. The client uses these to update the UI.

**Progress notification from the server:**

```json
{
  "jsonrpc": "2.0",
  "method": "notifications/progress",
  "params": {
    "progressToken": "tok-7",
    "progress": 60,
    "total": 100,
    "message": "Searching 600 of 1000 files..."
  }
}
```

| Field           | Description                               |
| --------------- | ----------------------------------------- |
| `progressToken` | Links this update to the original request |
| `progress`      | Current progress value                    |
| `total`         | Total expected value (when known)         |
| `message`       | Human-readable status message             |

---

## 🗺️ Full Lifecycle Diagram

```
Client                                          Server
  │                                               │
  │──── initialize (protocolVersion, caps) ──────▶│
  │                                               │
  │◀─── result (protocolVersion, caps) ───────────│
  │                                               │
  │──── notifications/initialized ───────────────▶│
  │                                               │
  │          [ OPERATION PHASE ]                  │
  │                                               │
  │──── tools/list ──────────────────────────────▶│
  │◀─── result (tools[]) ─────────────────────────│
  │                                               │
  │──── tools/call (name, args, _meta) ──────────▶│
  │◀─── notifications/progress (60/100) ──────────│
  │◀─── result (content) ─────────────────────────│
  │                                               │
  │          [ SHUTDOWN PHASE ]                   │
  │                                               │
  │──── [close stream / HTTP connection] ────────▶│
  │◀─── [server exits] ───────────────────────────│
```

---

_Based on the MCP Protocol Specification — `protocolVersion: 2025-03-26`_
