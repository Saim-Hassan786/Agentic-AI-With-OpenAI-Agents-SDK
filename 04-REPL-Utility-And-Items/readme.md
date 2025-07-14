 # 🧪 Run-Demo-Loop in OpenAI Agents SDK

The `run_demo_loop` is a **built-in testing utility** provided by the OpenAI Agents SDK that allows developers to **manually test and interact with an agent in a loop** through a simple command-line interface. It is especially useful in development and debugging phases of agent creation, helping to quickly validate the behavior of agents before deploying them in production workflows.


## 🧠 What is `run_demo_loop`?

`run_demo_loop` is a **testing loop function** that simulates an interactive environment where developers can send text inputs to an agent and receive real-time outputs, without writing boilerplate code to run the agent manually.

It leverages the internal `Runner` logic of the SDK to:

* Create a consistent, minimal environment to execute the agent.
* Allow user input to be directly passed as an `input` to the agent.
* Display the **final output** and intermediate steps clearly for inspection.


## 🎯 Key Goals

* ✅ **Fast Testing**: Quickly prototype and test an agent’s logic.
* ✅ **Real-Time Feedback**: View how an agent handles different queries.
* ✅ **Debugging Utility**: Observe intermediate decisions, tool calls, and reasoning.
* ✅ **No Frontend Required**: Use from terminal or notebook without a UI.


## ⚙️ How it Works

Internally, `run_demo_loop` sets up a basic runner call inside an input loop:

```python
await run_demo_loop(starting_agent)
```

* It waits for a **user input**.
* Wraps the input into a minimal `RunInput` structure.
* Passes it to the `Runner.run()` method.
* Displays the **final output**, and optionally intermediate tool calls or messages.

This loop continues until the user exits (e.g., typing `exit` or pressing `Ctrl+C`).


## 🧩 Use Cases

* 🧪 Testing custom agents during development.
* 🗣️ Debugging how an agent interprets different instructions.
* 🔍 Analyzing how guardrails, input parsing, and tool responses behave.
* 🧰 Tuning prompts, tools, and agent strategy by trial.


## 📦 Example Usage

```python
from agents import Agent, run_demo_loop

agent = Agent(name="AssistantAgent")

await run_demo_loop(agent)
```

This will start an interactive loop in your terminal/notebook where you can type messages like:

```text
You: What's the capital of Germany?
Agent: 🧠 Thinking... 🏙️ The capital of Germany is Berlin.
```


## 🧼 Notes

* `run_demo_loop` is for **manual agent evaluation** only, not production use.
* It’s ideal for **prototyping**, not load-testing or automated testing.
* Make sure your agent and tools are fully registered and initialized before passing to the loop.


## 📖 Summary

The `run_demo_loop` is a **developer-friendly**, no-setup-needed utility in the OpenAI Agents SDK to **interactively test agents** via terminal or notebook. It encourages rapid iteration, learning, and debugging, making it an essential tool when building custom autonomous agents.

