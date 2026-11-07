# Autonomous Enterprise Multi-Agent System with MCP & LangGraph

An autonomous multi-agent engineering workflow built using **LangGraph** and Anthropic's **Model Context Protocol (MCP)**. The system orchestrates specialized reasoning agents (Orchestrator, Planner, Coder) connected to an isolated MCP tool gateway to plan, generate, and isolate code implementations across independent branches.

---

## Architecture Overview

Traditional ReAct agent loops degrade in accuracy when overloaded with dozens of disparate tools. This architecture uses a **Supervisor-Worker pattern** coupled with a dedicated **MCP Gateway Server** to enforce strict interface contracts, isolate tool contexts, and manage state transitions deterministically.

```mermaid
graph TD
    User([Human Operator / User]) -->|Input Task / Feature Request| Orchestrator[Orchestrator Agent]
    
    subgraph Multi-Agent Graph [LangGraph State Machine]
        Orchestrator -->|Delegates Strategy| Planner[Planner Agent]
        Planner -->|Returns Multi-Approach Plan| Orchestrator
        Orchestrator -->|Dispatches Specs| Coder[Coder Agent]
        Coder -->|Reports Commits / PR Status| Orchestrator
    end

    subgraph Tool Infrastructure
        Orchestrator -.->|JSON-RPC Protocol| Gateway[MCP Gateway Server]
        Coder -.->|JSON-RPC Protocol| Gateway
        Gateway --> FilesystemMCP[Filesystem MCP Server]
        Gateway --> GitHubMCP[GitHub / Git MCP Server]
        Gateway --> MemoryMCP[Knowledge Graph Memory MCP]
    end

    Coder -->|Pushes Branches| GitHubRemote[GitHub Repository]
