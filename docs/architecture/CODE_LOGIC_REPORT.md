# Hermes Agent: Code Logic & Patterns Report

## 1. Implementation Deep Dives

### 1.1. Agent Loop Lifecycle (`run_agent.py`)
The `AIAgent.run_conversation` logic follows this strict order:
1.  **Hydration**: Restores Todo state and nudge counters from session history.
2.  **System Prompt Assembly**: Rebuilds or reloads the cached system prompt.
3.  **Preflight Compression**: Checks context window limits before the first API call.
4.  **Plugin Hooks**: Fires `pre_llm_call`.
5.  **Memory Prefetch**: External memory providers are queried once per turn.
6.  **The Loop (`while`)**:
    -   Checks for user interrupts.
    -   Sanitizes tool call arguments and message sequence (alternation).
    -   Injects ephemeral context (memory, plugins) into the user message.
    -   Executes the API call (with extensive retry/fallback logic).
    -   Dispatches tool calls to the Registry.
    -   Accumulates trajectory data.
7.  **Finalization**: Fires `post_llm_call`, updates Todo/Skill counters, and persists the session.

### 1.2. Tool Dispatch Pipeline
When the LLM issues a tool call:
1.  `AIAgent` identifies the tool (Built-in, Context Engine, or Memory).
2.  `model_tools.handle_function_call` is called.
3.  **Hooks**: `pre_tool_call` plugin hooks are checked.
4.  **Registry Dispatch**: `registry.dispatch` retrieves the `ToolEntry`.
5.  **Execution**: The handler is executed (bridged to `_run_async` if necessary).
6.  **Post-Process**: `post_tool_call` and `transform_tool_result` hooks are fired.
7.  **Return**: The JSON-serialized result is returned to the agent loop.

### 1.3. Messaging Adapter Lifecycle
1.  `GatewayRunner` instantiates adapters based on `config.yaml`.
2.  `adapter.start()` initializes the platform connection (e.g., Long Polling or Webhook).
3.  `adapter.handle_message(MessageEvent)` is triggered on inbound traffic.
4.  **Queueing**: Messages are queued if the session is already busy.
5.  **Execution**: `GatewayRunner._handle_message` runs the agent in a background task.
6.  **Streaming**: Tools or LLM deltas are streamed back to the platform via `adapter.send_draft` or `send_typing`.

## 2. Analyzed Patterns

| Pattern | Usage in Hermes |
| :--- | :--- |
| **Registry** | Centralized management of tools, commands, and providers to avoid hardcoded imports. |
| **Adapter** | Normalizing heterogeneous interfaces (LLM providers, Messaging platforms, Terminal backends). |
| **Strategy** | Pluggable logic for memory recall, context compression, and image generation. |
| **Middleware / Hooks** | Plugin system allows injecting logic at specific execution points without modifying core files. |
| **Proxy / RPC** | Used in TUI architecture to separate the Node.js UI from the Python agent logic. |
| **Repository** | `SessionDB` abstracts session persistence using SQLite/FTS5. |

## 3. Dependency Injection Patterns
Hermes primarily uses **Manual Dependency Injection** via constructor parameters (e.g., `AIAgent.__init__` takes 60+ arguments). While this makes dependencies explicit, it contributes to "God Object" complexity. The system also uses **Service Locator** patterns via module-level registries (e.g., `from tools.registry import registry`).

## 4. Module Loading System
The system uses a mix of static imports and dynamic `importlib`-based discovery.
-   **Static**: Core orchestration (`run_agent`, `cli`, `gateway`).
-   **Dynamic**: Tools (`tools/registry.py`), Plugins (`hermes_cli/plugins.py`), and Model Providers (`plugins/model-providers/`).
-   **Lazy**: Heavy dependencies are often imported inside functions or via `tools/lazy_deps.py` to keep startup fast.
