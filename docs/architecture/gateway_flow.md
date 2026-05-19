# Gateway Message Flow

## Mermaid.js Diagram
```mermaid
sequenceDiagram
    participant P as Platform (Telegram/Discord/etc)
    participant AD as Adapter
    participant GR as GatewayRunner
    participant A as AIAgent

    P->>AD: Inbound Message
    AD->>GR: _handle_message(MessageEvent)

    rect rgb(200, 220, 240)
        Note over GR: Gateway Processing
        GR->>GR: Check Auth / Pairing
        GR->>GR: Intercept Slash Commands
        GR->>GR: Check Active Session (Interrupt?)
    end

    GR->>A: run_conversation(text)

    loop Agent Loop
        A-->>GR: status / tool_progress events
        GR-->>AD: send_typing / send_draft
    end

    A-->>GR: final_response
    GR-->>AD: send(chat_id, response)
    AD-->>P: Outbound Message
```

## ASCII Diagram
```text
  PLATFORM              ADAPTER             GATEWAY RUNNER             AI AGENT
      |                    |                      |                        |
      |--- Inbound Msg --->|                      |                        |
      |                    |--- _handle_message ->|                        |
      |                    |                      |                        |
      |                    |             +------------------+              |
      |                    |             | 1. Auth/Pairing  |              |
      |                    |             | 2. Slash Cmds    |              |
      |                    |             | 3. Interrupts    |              |
      |                    |             +------------------+              |
      |                    |                      |                        |
      |                    |                      |--- run_conversation -->|
      |                    |                      |                        |
      |                    |             [ Agent Execution Loop ]          |
      |                    |                      |<-- status/progress ----|
      |                    |<--- send_typing -----|                        |
      |                    |                      |                        |
      |                    |                      |<--- final_response ----|
      |                    |<--- send(resp) ------|                        |
      |<--- Outbound Msg --|                      |                        |
      |                    |                      |                        |
```
