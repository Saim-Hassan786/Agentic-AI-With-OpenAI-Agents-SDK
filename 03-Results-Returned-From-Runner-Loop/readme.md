# 🧠 Understanding `RunResult` and `RunResultStreaming` in OpenAI Agents SDK

The OpenAI Agents SDK allows you to build **agentic applications** using the `Runner` abstraction. After executing an agent via the `Runner`, the SDK provides structured **result objects** that capture everything that happened during the agent’s execution lifecycle.

This README explains the **two result objects**:

* `RunResult` (used in standard agent runs)
* `RunResultStreaming` (used in streamed runs)

Both serve as **introspective artifacts** that help you debug, inspect, or further process what the agent did and said.


## 🔁 When Are These Returned?

* `RunResult` is returned when you use `await Runner.run(...)` or `Runner.run_sync(...)`.
* `RunResultStreaming` is returned when you use `await Runner.run_streamed(...)`.



## 📦 1. `RunResult`: Final Result of an Agent Run from Runner.run or run_sync

The `RunResult` class encapsulates the complete outcome of a full agent execution.

### ✅ Primary Use Case:

* When you want to wait for the agent to complete before accessing the result.
* Useful in APIs, testing, and non-interactive environments.


### 🧠 Conceptual Role:

`RunResult` acts as a **transcript of the agent's entire thinking and action process**, from input to final answer. You can inspect all intermediate steps, tools used, LLM generations, and guardrails triggered.


## 🌊 2. `RunResultStreaming`: Real-Time Streaming Result Object

`RunResultStreaming` allows you to **consume partial outputs as they are produced** by the agent.

### ✅ Primary Use Case:

* Real-time UIs (e.g., chat apps).
* Voice agents, CLI interfaces, or progressive rendering.
* When you want output tokens or tool calls streamed incrementally.


### 🧠 Conceptual Role:

`RunResultStreaming` lets you **observe and interact** with an agent **while it’s still thinking and generating**. It is ideal for cases where responsiveness and latency are important.


## 🆚 `RunResult` vs `RunResultStreaming`

| Feature              | `RunResult`             | `RunResultStreaming`     |
| -------------------- | ----------------------- | ------------------------ |
| Full Output          | At end                  | Streams during execution |
| Real-time Feedback   | ❌ No                    | ✅ Yes                    |
| Suitable for Chat UI | ⚠️ Less ideal           | ✅ Perfect                |
| Abortable            | ❌ No                    | ✅ Yes                    |
| Error Tracking       | Final only              | As it happens            |
| Use With             | `run()` or `run_sync()` | `run_streamed()`         |


## 📚 Conclusion

The `RunResult` and `RunResultStreaming` classes provide **introspective, debuggable, and structured representations** of what the agent did — either after finishing the task (`RunResult`) or while it’s doing it (`RunResultStreaming`).

These objects are **central to building reliable, observable, and interactive agent experiences** using the OpenAI Agents SDK.
