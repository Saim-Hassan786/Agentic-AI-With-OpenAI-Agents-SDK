# 🏃 Runner Class – OpenAI Agents SDK

The **`Runner` class** in the OpenAI Agents SDK is the core orchestrator that runs **agent workflows**, managing interactions with LLMs, tool calls, agent handoffs, streaming, and lifecycle events.



## 🔄 1. Core Functionality

`Runner.run()` (async) and `Runner.run_sync()` (sync wrapper) execute a workflow:

1. Call LLM using the current agent and input.
2. If agent returns a **final output**, the run ends.
3. If agent triggers a **handoff**, switch to a new agent and continue.
4. Otherwise, if there are **tool calls**, execute them, append results, and continue.
5. Repeat until completion or until `max_turns` is exceeded.
   

## ⚙️ 2. API Methods

### `run(starting_agent, input, context=None, max_turns=10, hooks=None, run_config=None, previous_response_id=None)`

* `starting_agent`: Entry-point Agent.
* `input`: User input (string or list of items).
* `context`: Optional app-specific object passed through the workflow. 
* `max_turns`: Maximum LLM invocation cycles.
* `hooks`: Agent lifecycle hooks (start, tool call, handoff, end).
* `run_config`: Global overrides (model settings, guardrails, tracing).
* `previous_response_id`: Use for resuming in OpenAI Responses API. 

Returns a `RunResult`.



### `run_sync(...)`

Synchronous version of `run()` for non‑async contexts. 



### `run_streamed(...)`

Async streaming version returning a `RunResultStreaming`, which emits structured semantic events as the agent works. Streams include agent updates, tool usage, and final output. 



## 🧩 3. Execution Flow

This is the internal agent loop:

1. **Invoke LLM** — Submit messages, possibly with system, tool instructions, and history.
2. **LLM Outputs**:

   * *Final output* → stops loop.
   * *Tool call* → trigger tool, append output, loop.
   * *Handoff* → switch agent, loop.
     Robust loop ensures any combination of tool usage and delegation is handled.

### ❗ Errors:

* `MaxTurnsExceededError` if loop exceeds `max_turns`.
* `GuardrailTripwireTriggered` if any guardrails fail.



## 🧰 4. RunConfig: Global Control

`RunConfig` (in `run_config=`) provides:

* `model`: Force global model override.
* `model_provider`: Customize source (e.g., Groq, OpenAI).
* `model_settings`: Control temperature, max tokens.
* `handoff_input_filter`: Apply filters to every handoff input.
* `input_guardrails` / `output_guardrails`: Validate inputs/outputs.
* `tracing_disabled`: Toggle internal tracing.
* `trace_include_sensitive_data`, `workflow_name`, etc. 



## 🎙️ 5. Context & Hooks

### Context

Optional data store (e.g. user metadata, DB connections) passed to all agents and tools. Very useful for dependency injection. 

### Hooks

Implement `AgentHooks` to tap into lifecycle events:

* `on_start`
* `on_end`
* `on_tool_start` / `on_tool_end`
* `on_handoff`

Useful for logging, monitoring, side-effects.



## 📊 6. Streaming Mode

Use `run_streamed()` for real-time output. Results can be consumed via `.stream_events()` — ideal for progressive UI or live updates.



## 🧩 7. Why Runner Matters

* **Simplifies agent orchestration** — abstracts tool use, delegation, termination.
* **Integrates tracing & guardrails** — for safe, provable workflows.
* **Supports complex modular systems** — with multiple agents and tools.
* **Provider-agnostic** — supports many LLM backends.



## ✅ Example (Python)

```python
from agents import Agent, Runner

agent = Agent(name="Assistant", instructions="You are helpful.")
result = Runner.run_sync(agent, "What is the capital of Sweden?", max_turns=5)
print(result.final_output)
```



## ✅ Summary

The `Runner` class is **the backbone** of agentic workflows:

* Handles LLM calls, tool usage, agent handoffs
* Enforces guardrails, tracing, lifecycle hooks
* Offers both sync and streaming APIs
* Globally configurable via `RunConfig`

Ideal for building **multi-agent, modular AI systems**.
