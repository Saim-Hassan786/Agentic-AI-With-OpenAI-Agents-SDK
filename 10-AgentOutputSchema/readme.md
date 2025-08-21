# 📘 Understanding `AgentOutputSchema`

This project demonstrates how to use **`AgentOutputSchema`** from the [`openai-agents`](https://pypi.org/project/openai-agents/) SDK to enforce structured output from LLMs. The goal is to ensure that the model’s responses strictly match a schema you define, using **Pydantic models** for validation.


## 🚀 Key Concepts

### 🔹 1. Agent and Output Type

An `Agent` represents a conversational AI powered by an LLM (here, **Gemini-2.5-Flash**).

* Each `Agent` has:

  * **name** → Identifier for the agent.
  * **instructions** → System-level prompt guiding the agent.
  * **model** → The LLM backend.
  * **output\_type** → Defines the expected format of the agent’s output.

The **`output_type`** is crucial:

* If it’s a string → the agent outputs plain text.
* If it’s a **Pydantic model** → the agent must respond in structured JSON matching the model’s schema.
* If it’s wrapped in **`AgentOutputSchema`** → stricter schema enforcement and validation apply.


### 🔹 2. `AgentOutputSchemaBase`

`AgentOutputSchemaBase` is the abstract base for schema handling. It provides:

* `is_plain_text()` → checks if schema expects free text.
* `json_schema()` → generates the JSON schema for the LLM.
* `is_strict_json_schema()` → enforces strict validation (no extra fields allowed).
* `validate_json()` → validates LLM responses against the schema.
* `name()` → returns schema name.


### 🔹 3. Cases of `output_type`

#### 🟢 Case 1: `output_type = str` or `None`

* Model returns plain text only.
* No schema applied.
* No validation performed.

#### 🟡 Case 2: `output_type = Pydantic Model` or `AgentOutputSchema`

* A proper **JSON schema** is generated.
* LLM is instructed to respond in JSON.
* SDK validates and parses response into your model.
* Guarantees structured, predictable outputs.

#### 🔴 Case 3: `output_type = primitive type` (e.g., `int`, `list`, `float`)

* Primitives aren’t directly supported in JSON schema.
* SDK wraps them inside a dict like `{ "output": 123 }`.
* LLM responds in wrapped format → unwrapped after validation.


## 🧪 Code Walkthrough

### ✅ Example 1: Pydantic Output Type

```python
class OutputType(BaseModel):
    reasoning : str
    response : str
    is_response_correct : bool
```

* The agent returns a structured JSON with reasoning, response, and a correctness flag.
* Output is automatically validated and parsed into `OutputType`.


### ✅ Example 2: AgentOutputSchema

```python
class OutputType2(BaseModel):
    is_home_work_question : bool
    response : str
    is_response_correct : bool
```

* Here we wrap it with `AgentOutputSchema(output_type=OutputType2, strict_json_schema=True)`.
* This enforces **strict validation** → no missing/extra fields allowed.
* Example response:

  ```json
  {
    "is_home_work_question": false,
    "response": "The capital of France is Paris and the capital of Germany is Berlin.",
    "is_response_correct": true
  }
  ```

### ✅ Schema Introspection

You can directly inspect the schema:

```python
print(output_schema.is_plain_text())
print(output_schema.is_strict_json_schema())
print(output_schema.json_schema())
print(output_schema.name())
print(output_schema.validate_json(json_str=json_str))
```

This helps in debugging and verifying what the LLM is being asked to produce.

## 📊 Summary

* **Plain text outputs** are unstructured (Case 1).
* **Pydantic models** enforce structure (Case 2).
* **AgentOutputSchema** ensures **strict schema compliance** and validation.
* **Primitive types** are wrapped to maintain consistency (Case 3).


## 🎯 Why Use `AgentOutputSchema`?

* Ensures **reliable, structured responses** from LLMs.
* Prevents schema drift or unexpected formats.
* Enables downstream automation (agents, workflows) where strict types are required.
* Ideal for use cases like:

  * Homework/quiz detection.
  * JSON API responses.
  * Reasoning + final answer separation.
  * Any pipeline that depends on predictable output.

## ✅ Conclusion

`AgentOutputSchema` is a powerful way to **bridge the gap between flexible LLM outputs and strict application requirements**. By combining LLM reasoning with **typed validation**, you get reliable, structured, and machine-usable results.
enforcement → validation → final structured result, for the README?

