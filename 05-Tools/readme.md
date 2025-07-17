## 📌 What is a Tool?

In the OpenAI Agents SDK, a **tool is a callable Python function** that can be registered with an LLM agent. The agent can choose to invoke the tool when it determines that completing a task requires it.

Think of tools as:

- Extensions of the LLM’s capabilities
- Interfaces to external code and real-world logic
- Secure and structured functions that the agent can "call" with reasoning

Example:

```python
from openai import tool

@tool
def get_time(city: str) -> str:
    """Returns the current time in a given city."""
    ...
````

This function becomes a *tool* the agent can use if the user asks, "What time is it in Berlin?"


## 🧠 Why Tools Matter

Without tools, an LLM is limited to generating responses based solely on its pre-trained knowledge and the current input prompt.

With tools, an LLM agent becomes capable of:

* Performing real-time actions (e.g., fetch stock prices)
* Querying databases or APIs
* Doing math computations
* Searching documents or the internet
* Automating workflows or decisions

Tools unlock **agentic behavior**: the ability of the agent to act in the environment, gather data, and adapt its reasoning.


## ⚙️ How the `tool` Parameter Works Internally

When you create an agent or a model in the OpenAI Agents SDK, you pass a list of tools using the `tools` parameter:

```python
from openai import tool, Agent, OpenAIChatCompletionsModel

@tool
def add(a: int, b: int) -> int:
    """Adds two numbers"""
    return a + b

agent = Agent(
    name="Math Assistant",
    model=OpenAIChatCompletionsModel(
        model="gpt-4o",
        api_key="your-api-key",
        tools=[add]  # 👈 this is where tools are passed
    )
)
```

During a conversation loop, the agent:

1. Parses the user input and determines if a tool call is needed.
2. Selects a tool by name and generates arguments in JSON format.
3. Validates the arguments against the tool’s schema (defined by the SDK).
4. Executes the tool using Python runtime.
5. Feeds the result back into the agent loop for further reasoning or response.


## 📦 Tool Schema & Validation

All tools are automatically converted into **JSON schemas** using Python type hints and docstrings via Pydantic under the hood. This ensures:

* The agent knows the input/output structure
* Inputs are type-checked before execution
* Auto-documentation for LLM to understand tool usage

For example:

```python
@tool
def get_weather(city: str, unit: Literal["C", "F"] = "C") -> str:
    """Returns the current weather for a city."""
```

Is transformed into:

```json
{
  "name": "get_weather",
  "description": "Returns the current weather for a city.",
  "parameters": {
    "type": "object",
    "properties": {
      "city": {"type": "string"},
      "unit": {"type": "string", "enum": ["C", "F"]}
    },
    "required": ["city"]
  }
}
```

This is the schema the LLM sees and uses when deciding how to call the tool.


## 🧵 Tools in Streamed Agents

When using `run_streamed()` or `RunResultStreaming`, tool usage happens in **real-time streaming steps**. Each phase of the tool call (start, function call, function output, finish) is sent as a streaming event:

* `on_tool_started`
* `on_tool_call_created`
* `on_tool_result`
* `on_tool_finished`

This provides a responsive, interactive experience where the agent can narrate its thought process.


## 🧪 Tool Types

There are two primary ways to define tools:

### 1. `@tool` Decorator (Simple Tool)

Basic tool registration using type hints.

```python
@tool
def translate(text: str, to_lang: str) -> str:
    """Translates text into a given language."""
```

### 2. `@function_tool` Decorator (Advanced Tool)

Used when you need more control, such as handling errors, timeouts, or async execution.

```python
from openai import function_tool

@function_tool(name="send_email")
async def send_email_async(recipient: str, subject: str, body: str):
    ...
```


## 🚨 Tool Call Errors and Safeguards

When the LLM tries to call a tool:

* If the tool is **not registered**, it fails
* If **invalid inputs** are passed (wrong types), validation fails
* If the function **raises an exception**, the agent handles the error gracefully

This protects the system from bad model behavior or user injection.

You can also define fallback/error handlers for more control.


## 🚀 Best Practices

| ✅ Do                     | 🚫 Avoid                          |
| ------------------------ | --------------------------------- |
| Use precise type hints   | Accepting `dict` or `Any` blindly |
| Add clear docstrings     | Omitting parameter descriptions   |
| Keep tools deterministic | Adding randomness unless needed   |
| Handle edge cases        | Letting tools fail silently       |
| Limit side-effects       | Avoiding unsafe writes or actions |


## 🧠 Summary Table

| Feature      | Description                                                           |
| ------------ | --------------------------------------------------------------------- |
| Purpose      | Extend agent reasoning with external function execution               |
| Registration | `tools=[func1, func2]` or via `@tool` / `@function_tool`              |
| Input Schema | Auto-generated from type hints + docstrings using JSON Schema         |
| Validation   | Strict input validation using `pydantic`                              |
| Streaming    | Supported with event lifecycle (tool\_started, tool\_result, etc.)    |
| Use Cases    | APIs, databases, math, file I/O, external logic, real-time operations |


## 📚 Further Reading

* [Agents SDK Docs](https://github.com/openai/openai-python/blob/main/openai/agents/README.md)
* [Function Calling in OpenAI Models](https://platform.openai.com/docs/guides/function-calling)
* [Pydantic Models](https://docs.pydantic.dev)


By using tools, you're enabling the LLM to **observe, reason, and act** — the core loop of autonomous agentic systems. Tools are the bridge between language understanding and real-world capability.

