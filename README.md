<div align="center">
  <img src="logo.jpg" alt="geminiFlow" width="350">
</div>

**geminiMind** is a foundational setup for reliable, agentic [Gemini CLI](https://geminicli.com/) workflows. It provides a structured environment and pre-configured tools to jumpstart your AI-assisted development.

> [!TIP]
> **Start Here**: For deep operational details and the core workflow philosophy, please read [GEMINI.md](.gemini/GEMINI.md) first.

---

## 💠 Workflow Diagram

This unified diagram represents the entire geminiFlow architecture: from the **Boot Sequence** that initializes the environment, to the **Operational Loop** that drives the agent, and finally the **Task Protocol** that executes specific work phases.

```mermaid
flowchart TD
    %% Styling
    classDef file fill:#f0fdf4,stroke:#22c55e,stroke-width:2px,color:#000,stroke-dasharray: 5 5
    classDef process fill:#eff6ff,stroke:#3b82f6,stroke-width:2px,color:#000
    classDef decision fill:#fefce8,stroke:#eab308,stroke-width:2px,color:#000
    classDef stop fill:#fee2e2,stroke:#ef4444,stroke-width:2px,color:#000
    classDef phase fill:#f3e8ff,stroke:#a855f7,stroke-width:2px,color:#000

    %% --- 0. BOOT SEQUENCE ---
    subgraph Boot ["🚀 0. BOOT_SEQUENCE"]
        direction TB
        Start((Start)) --> Init{Has $MB?}:::decision
        Init -->|No| Setup(1. INIT\nMkdir & Copy Tpl):::process
        Init -->|Yes| Load(2. LOAD\nRead Variables):::process
        Setup --> Load
        Load --> Align(3. ALIGN\nRead Active Task):::process
    end

    %% --- 2. OPERATIONAL LOOP ---
    subgraph Loop ["⚙️ 2. OPERATIONAL LOOP"]
        direction TB
        Sync(1. SYNC\nRead $CTX & Phase):::process
        Strategy{2. STRATEGY}:::decision
        Stop([STOP\nAmbiguous/Done]):::stop
        Persist(3. PERSIST\nUpdate $CTX):::process

        Align --> Sync
        Sync --> Strategy
        Strategy -->|Ambiguous| Stop
        Persist -->|Next Tick| Sync
    end

    %% --- 1. TASK PROTOCOL ---
    subgraph Protocol ["1. TASK_PROTOCOL (Execution)"]
        direction TB
        CheckPhase{Check Phase}:::decision
        
        %% Phases
        Disc(DISCOVERY\nmap, find_code):::phase
        Spec(SPEC\nplan, query-docs):::phase
        Impl(IMPL\nTDD loop, write):::phase
        Verify(VERIFY\ntest, debug):::phase
        
        Strategy -->|Execute| CheckPhase
        CheckPhase --> Disc & Spec & Impl & Verify
        
        %% Transitions
        Disc -.-> Spec -.-> Impl -.-> Verify
        
        %% Completion
        IsDone{Task Done?}:::decision
        Archive(🏁 COMPLETE\nMove to /completed):::process
        
        Verify --> IsDone
        IsDone -->|No| Persist
        IsDone -->|Yes| Archive --> Stop
    end

    %% Data Stores
    Files[("💾 File System\n$MB (Memory)\n$TASKS (Stories)\n$CTX (State)")]:::file
    Files -.- Load
    Files -.- Sync
    Files -.- Persist
```

---

## 🚀 Getting Started

### Prerequisites

- **Node.js & NPM**: Required for the `context7` MCP server.
- **Python (uv recommended)**: Required for the `ast-grep` MCP server.

### Installation

1. Clone this repository.
2. Review the configuration files in `.gemini/`.
3. Configure your MCP servers as described below.

---

## 🧑‍💻 Common Prompts

```bash
INIT $MB -- PROJECT: Social Network for Rabbits
```

```bash
LOAD_FULL $MB -- THEN_CREATE $TASKS: Example Feature Request
```

```bash
EXECUTE $TASKS
```

---

## ⚙️ Configuration & MCP Servers

The core configuration lives in [`.gemini/settings.json`](.gemini/settings.json). This file pre-configures two powerful Model Context Protocol (MCP) servers.

### 1. [@upstash/context7-mcp](https://github.com/upstash/context7)

Up-to-date lib/dependency documentation for Agents.

- **Status**: Automatic Installation

- **Action Required**: Add your API Key.
This server will automatically install via `npx` on systems with NPM available. You simply need to open `settings.json` and replace `KEY_HERE` with your actual API key.

### 2. [ast-grep-mcp](https://github.com/ast-grep/ast-grep-mcp)

Focused structural code search for Agents.

- **Status**: Manual Installation Required

- **Action Required**: Install server & update path.
This server provides structural code search capabilities. You must install it manually on your machine.

1. Follow the installation instructions at: [https://github.com/ast-grep/ast-grep-mcp](https://github.com/ast-grep/ast-grep-mcp)
2. Locate where the server was installed (e.g., path to `main.py`).
3. Update specific path in [`.gemini/settings.json`](.gemini/settings.json) to point to your local installation.

---

## 📂 Directory Structure

The `.gemini/` directory is the brain of this workflow:

- **`GEMINI.md`**: The master system prompt and operational protocol.
- **`settings.json`**: Tool and server configurations.
- **`templates/`**: Standardized file templates for consistency.
