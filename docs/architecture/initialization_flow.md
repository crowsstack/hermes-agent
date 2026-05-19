# Runtime Initialization Lifecycle

```text
1. [ hermes ] (cli.py / main.py)
   |
   +-- 2. _apply_profile_override()
   |      (Sets HERMES_HOME based on -p/--profile)
   |
   +-- 3. load_config()
   |      (Merges DEFAULT_CONFIG + ~/.hermes/config.yaml)
   |
   +-- 4. load_hermes_dotenv()
   |      (Loads API keys from ~/.hermes/.env)
   |
   +-- 5. discover_plugins()
   |      (Scans bundled, user, and pip plugins)
   |
   +-- 6. discover_builtin_tools()
   |      (Populates the Tool Registry via AST scanning)
   |
   +-- 7. AIAgent.__init__(...)
   |      (Resolves provider, model, and active toolsets)
   |
   +-- 8. run_conversation() / run_job() / start()
          (Enters the execution loop)
```
