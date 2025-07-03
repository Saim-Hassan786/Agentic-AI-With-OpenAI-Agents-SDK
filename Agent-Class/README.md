# OpenAI Agents SDK: Agent Class

## Agent Class Overview

The `Agent` class is the core building block of the OpenAI Agents SDK, encapsulating a Large Language Model (LLM) with specialized capabilities, tools, and behaviors. Each agent functions as an autonomous reasoning unit that can process inputs, make decisions, use tools, and collaborate with other agents.

```python
from agents import Agent

my_agent = Agent(
    name="Research Assistant",
    instructions="You help researchers find scientific papers",
    model="gemini-2.5-flash",
    tools=[search_tool, analyze_tool],
    # ... other parameters
)
```

## Agent Parameters Reference

### Core Configuration

1. **`name: str`**  
   A human-readable identifier for the agent used in logs, traces, and UI elements.  
   *Example:* `"Customer Support Agent"`

2. **`instructions: str | Callable`**  
   The system prompt guiding the agent's behavior. Can be dynamic:  
   ```python
   def dynamic_instructions(context, agent):
       return f"Help user in {context['language']} language"
   
   Agent(instructions=dynamic_instructions, ...)
   ```

3. **`model: str | Model`**  
   Specifies the LLM implementation:  
   - String model ID: `"gpt-4-turbo"`, `"gemini-2.5-flash"`  
   - Custom `Model` instance for advanced configuration  
   *Default:* SDK-configured default model

---

### Capability Extensions

4. **`tools: list[Tool]`**  
   Functions the agent can execute during reasoning:  
   ```python
   from agents import function_tool
   
   @function_tool
   def search_papers(query: str) -> list[dict]:
       "Search academic databases"
       return db.search(query)
   
   Agent(tools=[search_papers], ...)
   ```

5. **`handoffs: list[Agent] | list[Handoff]`**  
   Other agents this agent can delegate to:  
   ```python
   billing_agent = Agent(name="Billing Specialist", ...)
   
   Agent(
       handoffs=[billing_agent],
       handoff_description="Delegate payment issues"
   )
   ```

6. **`handoff_description: str`**  
   Documents when to use this agent's handoff capability  
   *Example:* "For technical support escalation"

---

### Behavior Control

7. **`model_settings: ModelSettings`**  
   Fine-tunes LLM behavior:  
   ```python
   from agents import ModelSettings
   
   ModelSettings(
       temperature=0.3,          # 0-2, creativity control
       max_tokens=2000,          # Response length limit
       top_p=0.9,                # Nucleus sampling
       tool_choice="auto",       # "required", "none"
       response_format="json"     # Force JSON output
   )
   ```

8. **`tool_use_behavior: str | CustomBehavior`**  
   Controls tool invocation strategy:  
   - `"run_llm_again"`: Continue after tools (default)  
   - `"stop_on_first_tool"`: Return immediately after tool call  
   - `StopAtTools`: Return when specific tools are called  
   - Custom function for advanced logic

9. **`reset_tool_choice: bool = True`**  
   Reset tool-selection state each turn to prevent repetitive tool calls

---

### Safety & Validation

10. **`input_guardrails: list[InputGuardrail]`**  
    Validate/sanitize inputs **before** processing:  
    ```python
    @input_guardrail
    async def profanityfilter(self, input: str, context) -> str:
            return profanity.censor(input)
    
    Agent(input_guardrails=[profanityfilter])
    ```

11. **`output_guardrails: list[OutputGuardrail]`**  
    Validate/transform outputs **after** generation:  
    ```python
     @output_guardrail
     async def output_guardrail_filter(self, output: str, context) -> str:
            return redact(output)
    
    Agent(output_guardrails=[output_guardrail_filter])
    ```

12. **`output_type: Type | Schema`**  
    Enforces response structure:  
    ```python
    from pydantic import BaseModel
    
    class WeatherResponse(BaseModel):
        temp: float
        conditions: str
    
    Agent(output_type=WeatherResponse)
    ```

---

### Advanced Features

13. **`hooks: AgentHooks`**  
    Lifecycle callbacks for custom logic:  
    ```python
    class LoggingHooks(AgentHooks):
        async def on_tool_start(self, tool, args):
            print(f"Starting {tool.name} with {args}")
    
    Agent(hooks=LoggingHooks())
    ```

14. **`mcp_servers: list[MCPServer]`**  
    Connect to external tool servers:  
    ```python
    MCPServer(
        url="https://tools.example.com",
        auth=ApiKeyAuth("sk-...")
    )
    ```

15. **`mcp_config: MCPConfig`**  
    Configuration for Managed Compute Platform:  
    ```python
    MCPConfig(
        timeout=30,
        retries=3,
        cache_enabled=True
    )
    ```

---

## Agent Lifecycle

1. **Initialization**  
   Agents are configured once and reused across sessions

2. **Execution Flow**  
   ```mermaid
   sequenceDiagram
       participant U as User
       participant R as Runner
       participant A as Agent
       
       U->>R: Input
       R->>A: Process input
       A->>A: Run input guardrails
       A->>LLM: Generate response
       alt Tool call
           A->>Tool: Execute
           Tool-->>A: Result
           A->>A: Decide next step
       end
       alt Handoff
           A->>R: Delegate to new agent
       else
           A->>R: Return final output
       end
       R->>A: Run output guardrails
       R-->>U: Final response
   ```

---

## Example Agent Configuration

```python
from agents import Agent, function_tool
from pydantic import BaseModel

# Define tools
@function_tool
def get_weather(location: str) -> dict:
    "Get current weather conditions"
    return weather_api(location)

@function_tool
def send_email(to: str, body: str) -> bool:
    "Send email notification"
    return email_client.send(to, body)

# Define output schema
class WeatherAlert(BaseModel):
    location: str
    temperature: float
    alert: str

# Create agent
weather_agent = Agent(
    name="Weather Alert Agent",
    instructions="""
    You monitor weather conditions and send alerts when 
    temperatures exceed 35°C. Format responses as JSON.
    """,
    model="gemini-2.5-flash",
    tools=[get_weather, send_email],
    output_type=WeatherAlert,
    tool_use_behavior="stop_on_first_tool",
    input_guardrails=[LocationValidator()],
    output_guardrails=[TemperatureUnitConverter()]
)
```

This agent will:  
1. Validate input locations  
2. Get weather data  
3. Send alerts via email if needed  
4. Return structured WeatherAlert objects  
5. Stop immediately after tool execution

## 🌐 Further Reading

* Official SDK Reference: [https://openai.github.io/openai-agents-python/](https://openai.github.io/openai-agents-python/)


