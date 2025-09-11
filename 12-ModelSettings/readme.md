# ModelSettings in OpenAI Agents SDK

This folder explains **ModelSettings** in the [OpenAI Agents SDK](https://github.com/openai/openai-agents)  and how they work at different levels (Agent vs Runner).

## 🔑 What are ModelSettings?

**ModelSettings** are configuration parameters that control and fine-tune the behavior of the underlying LLM (Language Model).  
They act as instructions for how the model should generate responses.  

Examples of configurable parameters include:

- `temperature`: controls randomness/creativity in responses.
- `top_p`: nucleus sampling to filter token probabilities.
- `max_tokens`: maximum length of generated output.
- `tool_choice`: specifies if and how tools can be used.
- `parallel_tool_calls`: enables or disables multiple tool calls at once.
- `truncation`: handles input longer than the context limit.
- `reasoning`: special parameter for reasoning models.
- `include_usage`: whether to include token usage in responses.
- `extra_*`: flexible fields (`extra_query`, `extra_args`, `extra_body`, `extra_headers`) to pass custom data.


## ⚖️ Agent-Level vs Runner-Level ModelSettings

ModelSettings can be applied at **two levels**:

### 1. Agent-Level
- Passed when creating an **Agent**.
- Defines the model behavior for that specific agent.
- Example:  
  ```python
  agent = Agent(
      name="Agent with Model Settings",
      instructions="Reply to user queries based on your Model Settings",
      model="gemini-2.5-flash",
      model_settings=ModelSettings(
          temperature=1.7,
          top_p=0.9,
          max_tokens=5000,
          tool_choice="auto",
          parallel_tool_calls=True,
          truncation="auto",
          include_usage=True
      )
  )
  ````

### 2. Runner-Level

* Passed in the **Runner** execution call (`Runner.run`) using `RunConfig`.
* **Overrides all Agent-level ModelSettings**.
* This ensures that the runtime execution always uses the runner’s configuration, even if the agent has its own settings.
* Example:

  ```python
  result = await Runner.run(
      agent,
      input="Hi there, tell me about AI in 100 words",
      run_config=RunConfig(
          model_settings=ModelSettings(
              temperature=1.7,
              top_p=0.9,
              max_tokens=5000,
              tool_choice="auto",
              parallel_tool_calls=True,
              truncation="auto",
              include_usage=True
          )
      )
  )
  ```

## 📌 Key Principle

> **Runner-level ModelSettings always take precedence over Agent-level ModelSettings.**

This design ensures flexibility:

* Agents can define **default behaviors**.
* Runner can **override behavior dynamically** at execution time.


## 🛠️ Practical Example

### Agent with ModelSettings:

* Settings applied when agent is defined.
* Used if no Runner-level overrides are provided.

### Runner with ModelSettings:

* Settings applied only at runtime.
* Always **override** any agent-defined settings.

## ✅ Summary

* **ModelSettings** fine-tune the model’s behavior (creativity, reasoning, tools, response length, etc.).
* Can be defined at **Agent-level** (defaults) or **Runner-level** (runtime overrides).
* **Runner-level settings > Agent-level settings** (they always win).
* Provides flexibility to adjust model configuration without modifying the agent itself.


## 📚 References

* [OpenAI Agents SDK Documentation](https://github.com/openai/openai-agents)
* [OpenAI API Reference](https://platform.openai.com/docs/)

