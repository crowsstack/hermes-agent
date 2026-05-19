# Hermes Agent: Pluggability & Flexibility Guide

## 1. Current Extension Points

Hermes is designed to be extended without touching the core codebase (`run_agent.py`, `cli.py`, etc.).

### 1.1. Plugins (General)
The most powerful extension point. Create a directory in `~/.hermes/plugins/<name>/` with:
-   `plugin.yaml`: Manifest (name, version, hooks).
-   `__init__.py`: Containing a `register(ctx)` function.
-   **Capabilities**: Register hooks, tools, and CLI subcommands.

### 1.2. Model Providers
Add new LLM backends by creating a plugin in `plugins/model-providers/`.
-   Implement a `ProviderProfile` and call `providers.register_provider()`.
-   Supports custom auth, base URLs, and API modes.

### 1.3. Memory Providers
Standardize memory recall by implementing the `MemoryProvider` ABC in `agent/memory_provider.py`.
-   Useful for integrating third-party vector DBs or user-profiling services.

### 1.4. Skills
Skills are non-code extensions. Create a `SKILL.md` in `~/.hermes/skills/` to provide the agent with specialized instructions, context, and reference material.

---

## 2. Recommendations for Future Modularity

To evolve Hermes into a more flexible, marketplace-ready ecosystem, the following architectural shifts are recommended:

### A. Break Down "God Objects"
-   **Refactor `AIAgent`**: Extract logic into smaller, task-specific controllers (e.g., `ConversationManager`, `ToolOrchestrator`, `APIClient`).
-   **Refactor `HermesCLI`**: Move command handlers and UI widgets into a more modular component-based system.

### B. Standardize Interfaces (Contracts)
-   Move away from 60-parameter constructors. Use configuration objects or dependency injection containers.
-   Formalize the `Tool` and `Platform` interfaces using strict Pydantic models or Protocols to ensure schema-driven consistency.

### C. Dynamic Injectable Features
-   Instead of `toolsets.py` being a static file, move toolset definitions to a dynamic registry where plugins can contribute to existing toolsets (e.g., a "finance" plugin adding tools to the "web" toolset).

### D. Decouple Runtime Dependencies
-   Use an Event Bus for internal communication between subsystems (e.g., `Agent -> EventBus -> UI`). This would decouple the agent logic from the ` KawaiiSpinner` and other display-specific code.
-   Standardize the `Transport` layer to allow swapping between stdio, WebSockets, or gRPC for all gateways.

### E. Schema-Driven Orchestration
-   Move hardcoded tool orchestration (like special handling for `execute_code` or `delegate_task`) into the tool handlers themselves or a generic middleware pipeline.

### F. Internal Utility Decoupling
-   Move common utilities (logging, constants, paths) into a dedicated `hermes_core` package that can be used by both the agent and standalone plugin repositories without importing the entire `hermes-agent` codebase.

## 3. Roadmap to a Marketplace Ecosystem
1.  **Phase 1 (Interface Formalization)**: Define strict contracts for all extension points.
2.  **Phase 2 (Subsystem Decoupling)**: Extract memory, context, and orchestration into independent modules.
3.  **Phase 3 (External Plugin Support)**: Allow plugins to be installed from remote repositories or a central registry.
4.  **Phase 4 (Schema-Driven UI)**: Allow plugins to contribute structured UI components to the TUI/Dashboard via the RPC layer.
