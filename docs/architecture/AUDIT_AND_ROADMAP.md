# Hermes Agent: Architectural Audit & Roadmap

## 1. Architectural Audit

### 1.1. Strengths
-   **Highly Pluggable**: Almost every major capability (tools, memory, models, context) is pluggable via clear ABCs and Registries.
-   **Robust Error Recovery**: Extensive logic in `run_agent.py` handles common LLM failures (Unicode, corrupted JSON, role alternation) gracefully.
-   **Multi-Platform by Design**: The `Gateway` and `Adapter` layers successfully unify disparate messaging protocols.
-   **Profile Isolation**: Excellent support for multiple isolated instances using `HERMES_HOME`.

### 1.2. Architectural Debt & Weaknesses
-   **Monolithic Classes**: `AIAgent` and `HermesCLI` are extremely large (>10k LOC) and handle too many responsibilities (God Objects).
-   **Constructor Bloat**: The `AIAgent` constructor's 60+ parameters make it difficult to instantiate and test in isolation.
-   **Tight Coupling**: High coupling between the agent loop and the TUI-specific display logic (`KawaiiSpinner`).
-   **Hardcoded Orchestration**: Specialized tools like `execute_code` and `delegate_task` have hardcoded paths in the agent loop.
-   **Sync/Async Complexity**: Sync->Async bridging is complex and scattered across `model_tools.py` and `tools/registry.py`.

### 1.3. Scalability & Maintenance Risks
-   **Context Bloat**: As the number of tools and skills grows, the system prompt size increases, leading to higher latency and costs.
-   **Memory Management**: The "one-external-provider" limit is a current bottleneck for users wanting to combine multiple memory strategies.
-   **TUI/RPC Fragility**: The JSON-RPC communication between the Python sidecar and the Node.js TUI can be fragile and hard to debug.

## 2. Refactor Recommendations

### Short-Term
-   **Extract Tool Orchestrator**: Move tool-calling logic from `AIAgent` to a dedicated `ToolOrchestrator` class.
-   **Standardize Config**: Use a structured `AgentConfig` dataclass instead of passing 60 parameters to `AIAgent`.
-   **Unify Displays**: Move all UI/terminal output logic behind a `DisplayInterface` to decouple the agent from the CLI/TUI implementations.

### Long-Term
-   **Event-Driven Core**: Implement a internal event bus for subsystem communication.
-   **Domain-Driven Design (DDD)**: Reorganize the codebase into clear domains: `Agent`, `Tools`, `Storage`, `UI`, `Gateway`.
-   **Schema-Driven Registry**: Move all toolset and platform configurations into a unified, schema-validated registry system.

## 3. Roadmaps

### 3.1. Scalability Roadmap
-   **Dynamic Tool Loading**: Only load tool schemas into the system prompt when relevant (semantic tool retrieval).
-   **Distributed Execution**: Allow tool backends (terminal, browser) to run on remote worker nodes.
-   **Context Summarization 2.0**: Implement more advanced, lossless context compression strategies (e.g., hierarchical summarization).

### 3.2. Modularity Roadmap
-   **Package Core**: Publish the core agent logic as a standalone library (`hermes-core`).
-   **Plugin Marketplace**: Develop a system for discovering and installing community-contributed plugins and skills.
-   **Universal Gateway**: Standardize the adapter interface to allow third-party developers to add new messaging platforms as standalone plugins.

### 3.3. Plugin System Roadmap
-   **Sandboxed Plugins**: Run plugins in isolated environments to improve security.
-   **UI Extension Points**: Allow plugins to contribute custom panels, sidebars, and widgets to the Dashboard and TUI.
-   **Bidirectional Hooks**: Allow plugins to not just observe but also fully replace or intercept core agent actions.
