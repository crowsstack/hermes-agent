# Hermes Agent: Technical Architecture Report

## 1. Executive Summary
Hermes Agent is a highly modular, self-improving AI agent system designed for high flexibility and scalability. It leverages a robust tool registry, a sophisticated plugin architecture, and multiple interaction gateways (CLI, TUI, Messaging) to provide a unified agentic experience. This report provides a deep dive into the system's architecture, orchestration flows, and core design principles.

## 2. High-Level Architecture
The system is divided into four main layers:
1.  **Interaction Gateways**: CLI, Gateway (Messaging platforms), TUI (React/Ink), and ACP (Editor integration).
2.  **Core Orchestration**: `AIAgent` (`run_agent.py`) and `ModelTools` (`model_tools.py`).
3.  **Capability Providers**: Tool Registry, Plugin System, Skill System, and MCP (Model Context Protocol).
4.  **State & Data Persistence**: SQLite-backed session store, Memory Providers, and Context Engines.

### Component Relationship
Refer to `docs/architecture/high_level.md` for the Mermaid diagram.

## 3. Core Subsystems

### 3.1. AIAgent (The "Brain")
The `AIAgent` class in `run_agent.py` is the central coordinator. It manages the conversation lifecycle, turn-based tool calling, and integration with memory/context subsystems.
-   **Conversation Loop**: Implemented in `run_conversation` and the internal `_run` loop. It handles strict role alternation, error recovery (sanitization), and iteration budgeting.
-   **API Transports**: Supports standard OpenAI-compatible Chat Completions, Codex Responses (with reasoning), and native Anthropic Messages API.
-   **Interrupt Handling**: Thread-safe interrupt signaling allows users to stop the agent mid-turn across all gateways.

### 3.2. Tool Registry & ModelTools
Hermes uses a **Registry Pattern** for tool management.
-   **Discovery**: `tools/registry.py` uses `ast` parsing to discover tools at import time without executing all modules.
-   **Schema Isolation**: Tools define their own schemas, handlers, and availability checks.
-   **Orchestration**: `model_tools.py` provides a thin wrapper for schema aggregation and dispatch, ensuring sync/async bridging via persistent event loops.

### 3.3. Plugin System
The plugin system (`hermes_cli/plugins.py`) provides extensive lifecycle hooks:
-   **Hooks**: `pre_llm_call`, `post_tool_call`, `on_session_start`, etc.
-   **Injection**: Plugins can register new tools, CLI commands, and even messaging platforms.
-   **Isolation**: Plugins can be bundled, user-installed, or pip-installed.

### 3.4. Messaging Gateway
The Gateway (`gateway/run.py`) uses an **Adapter Pattern** to support 20+ platforms.
-   **Base Adapter**: Defines the contract for sending messages, typing indicators, and handling media.
-   **Gateway Runner**: Manages the lifecycle of all active adapters and routes inbound events through authorization and command interception before reaching the agent.

### 3.5. TUI Architecture
The modern TUI (`ui-tui/`) is a **React (Ink) + Python Sidecar** architecture.
-   **Frontend**: React (Ink) manages the terminal UI, transcript rendering, and user input.
-   **Backend**: `tui_gateway/server.py` runs as a JSON-RPC sidecar, providing session management and agent execution services over stdio.

## 4. Cross-Cutting Concerns
-   **Profiles**: Fully isolated environments (`HERMES_HOME`) allow multiple instances with separate configs, memories, and sessions.
-   **Security**: Includes command approval flows, secret redaction, and URL/path safety checks.
-   **Memory**: Orchestrated by `MemoryManager`, supporting various backends (Honcho, Mem0, etc.) via a provider ABC.

## 5. Architectural Patterns
-   **Registry**: Used for tools, commands, platforms, and providers.
-   **Adapter**: Used for LLM providers and messaging platforms.
-   **Strategy**: Used for memory providers and context engines.
-   **Observer (Hooks)**: Used for plugin-based extensions.
-   **Singleton (Soft)**: Managed via module-level global states in some components (e.g., `registry` instance).
