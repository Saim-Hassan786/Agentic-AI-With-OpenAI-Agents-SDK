# Guardrails in the OpenAI Agents SDK

## 1. Introduction

In the **OpenAI Agents SDK**, the `guardrails` parameter allows you to define **safety, compliance, and operational boundaries** for an agent.
Think of guardrails as **active safety rules** that inspect either:

* **Inputs** to an agent before any reasoning is done.
* **Outputs** from an agent before they are returned or passed to another system.

They ensure that the agent operates **within predefined constraints**, helping to:

* Reduce the risk of unsafe, irrelevant, or costly operations.
* Keep multi-agent workflows stable and policy-compliant.
* Prevent unnecessary compute usage on invalid requests.


## 2. Theoretical Role of Guardrails in Agentic Systems

In **agent theory**, guardrails function as **runtime validation layers** within the broader *Sense → Think → Act* loop:

1. **Sense (Input)**
   Guardrails act as a *gatekeeper* before the agent “thinks.” They can:

   * Block irrelevant or malicious user queries.
   * Enforce data format, language, or scope.
   * Prevent the agent from starting expensive computations on bad input.

2. **Think → Act → Output**
   After reasoning, output guardrails ensure the result:

   * Meets compliance and safety requirements.
   * Is in the correct structure for downstream systems.
   * Does not contain disallowed or harmful content.

In control system terms, guardrails are **hard constraints** enforced at runtime, as opposed to **soft guidance** like prompt instructions.

---

## 3. Types of Guardrails

| Type                  | Execution Point                                  | Use Cases                                                           | Exception Raised                   |
| --------------------- | ------------------------------------------------ | ------------------------------------------------------------------- | ---------------------------------- |
| **Input Guardrails**  | Before agent runs (only if first in the chain)   | Block unsafe queries, validate schema, enforce access rules         | `InputGuardrailTripwireTriggered`  |
| **Output Guardrails** | After agent finishes (only if last in the chain) | Filter disallowed content, ensure correctness, strip sensitive info | `OutputGuardrailTripwireTriggered` |


## 4. Guardrail Execution Flow

```
[User Request]
   ↓
(Input Guardrails) ── validate? ── No → Tripwire Exception → Abort
   ↓ Yes
[Agent Reasoning & Tool Use]
   ↓
(Output Guardrails) ── validate? ── No → Tripwire Exception → Abort
   ↓ Yes
[Final Response Delivered]
```


## 5. The `guardrails` Parameter in Practice

The `guardrails` parameter accepts **one or more guardrail functions**.
Each guardrail is a **pure function** that receives the current context and returns a **GuardrailFunctionOutput** object with:

* `output_info`: metadata, logging details, reasoning about decision.
* `tripwire_triggered` (boolean): `True` means execution must stop.

**Pseudocode Example:**

```python
from agents import GuardrailFunctionOutput

def check_for_profanity(context):
    if "badword" in context.input_text.lower():
        return GuardrailFunctionOutput(
            output_info="Blocked: contains profanity",
            tripwire_triggered=True
        )
    return GuardrailFunctionOutput(
        output_info="Input clean",
        tripwire_triggered=False
    )

agent = Agent(
    name="safe_agent",
    guardrails=[check_for_profanity]
)
```


## 6. Why Guardrails Matter in Multi-Agent Systems

In a distributed **DACA-like agentic AI system**:

* **Performance Control**
  Prevents wasteful calls to LLMs or APIs for invalid input.

* **Security & Compliance**
  Stops policy-breaking responses before they propagate through the system.

* **Robustness in Chains**
  In multi-agent pipelines, only the **entry** agent’s input guardrails and the **final** agent’s output guardrails run — avoiding unnecessary checks at each hop.


## 7. Design Principles & Best Practices

1. **Fail Fast**
   The earlier an invalid request is detected, the better.

2. **Specificity over Generality**
   Each guardrail should enforce one type of rule (e.g., profanity filter, PII remover) to improve maintainability.

3. **Transparency**
   Include `output_info` for debugging and audit logs.

4. **Modularity**
   Guardrails should be re-usable across agents.

5. **Layered Safety**
   Guardrails are one layer; combine with:

   * Prompt engineering
   * LLM moderation APIs
   * Tool permissioning


## 8. Limitations & Theoretical Considerations

* **Not foolproof**: Guardrails operate *before* and *after* reasoning, but cannot control *internal thought steps* mid-process.
* **Trade-offs**: Overly strict guardrails can reduce usability; too loose, and safety is compromised.
* **Latency impact**: Heavy computation inside guardrails can add response time.
* **Dynamic adaptation**: Some systems might benefit from adaptive guardrails that adjust rules based on session context.


## 9. Conceptual Analogy

Guardrails are like:

* **Airport security gates** for inputs — preventing dangerous items from entering.
* **Customs inspection** for outputs — ensuring only safe, allowed goods leave.


## 10. Summary Table

| Aspect                | Details                                                        |
| --------------------- | -------------------------------------------------------------- |
| **Purpose**           | Enforce safety, compliance, and structural rules at runtime    |
| **Execution Points**  | Before reasoning (input) / after reasoning (output)            |
| **Trigger Mechanism** | Exception stops further execution if tripwire is hit           |
| **Return Format**     | `GuardrailFunctionOutput` with info + trigger flag             |
| **Scope in Chains**   | First agent (input guardrails), last agent (output guardrails) |
| **Best Practice**     | Keep rules atomic, reusable, and transparent                   |


## 11. Further Reading

* **Documentation:** [OpenAI Guardrails Docs](https://openai.github.io/openai-agents-python/guardrails/)

