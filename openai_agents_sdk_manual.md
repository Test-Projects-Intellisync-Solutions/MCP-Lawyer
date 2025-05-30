# OpenAI Agents SDK: A Beginner-Friendly Manual

## 1. Introduction

### What is the OpenAI Agents SDK?
The OpenAI Agents SDK is a Python library designed to help developers build agentic AI applications. It provides a lightweight, easy-to-use package with a focus on simplicity and extensibility. The SDK aims to offer enough features to be valuable while keeping the number of core concepts small, making it quick to learn.

### Why use it? Key design principles and benefits:

*   **Simplicity and Power**: The SDK is built with two main design principles:
    1.  **Enough features to be worth using, but few enough primitives to make it quick to learn.**
    2.  **Works great out of the box, but you can customize exactly what happens.**
*   **Key Benefits**:
    *   **Built-in Agent Loop**: Handles the complexities of calling tools, sending results to the Large Language Model (LLM), and iterating until the LLM's task is complete.
    *   **Python-first**: Encourages using standard Python language features for orchestrating and chaining agents, minimizing the need to learn new, complex abstractions.
    *   **Handoffs**: A powerful feature for coordinating tasks and delegating responsibilities between multiple specialized agents.
    *   **Guardrails**: Allows running input validations and checks in parallel with agent operations, enabling early termination if checks fail.
    *   **Function Tools**: Easily convert any Python function into a tool that an agent can use. This includes automatic schema generation from type hints and Pydantic-powered validation for tool inputs.
    *   **Tracing**: Comes with built-in tracing capabilities to help visualize, debug, and monitor agent workflows. This is extensible and supports various external destinations like Logfire, AgentOps, etc.

## 2. Installation

To get started with the OpenAI Agents SDK, you can install it using pip:

```bash
pip install openai-agents
```

**Important**: To use the SDK with OpenAI models, you must have your `OPENAI_API_KEY` environment variable set. Your MCP Lawyer project likely already handles this via `app/config.py`, but it's a crucial prerequisite.

```bash
export OPENAI_API_KEY='your-api-key-here'
```

## 3. Core Concepts

Understanding these core components is key to using the SDK effectively.

### `Agent`
An `Agent` represents an AI entity that can perform tasks. You define an agent by giving it:
*   `name`: A descriptive name for the agent (e.g., "LegalResearcher", "DocumentDrafter").
*   `instructions`: A set of instructions or a system prompt that defines the agent's role, capabilities, and how it should behave (e.g., "You are a helpful assistant specialized in legal research.").

Example:
```python
from agents import Agent

research_agent = Agent(name="LegalResearcher", instructions="You are an expert in finding relevant case law.")
```

### `Runner`
The `Runner` is responsible for executing an agent to perform a task. You provide the agent and the user's input (prompt) to the runner.

Example (synchronous execution):
```python
from agents import Agent, Runner

assistant_agent = Agent(name="Assistant", instructions="You are a helpful assistant.")
user_query = "Explain the concept of 'res judicata'."

result = Runner.run_sync(assistant_agent, user_query)
print(result.final_output)
```
The SDK also supports asynchronous execution.

### The Agent Loop
This is a fundamental part of the SDK, though often abstracted away by the `Runner`. The agent loop manages the entire lifecycle of an agent's task execution:
1.  Sends the user's prompt (and conversation history) to the LLM.
2.  Receives the LLM's response, which might include a request to call a tool.
3.  If a tool call is requested:
    *   The SDK (or your custom logic) executes the tool/function.
    *   The result from the tool is sent back to the LLM.
4.  The LLM processes the tool's result and continues, potentially requesting more tool calls or providing a final answer.
5.  This loop continues until the LLM indicates it has completed the task.

## 4. Key Features Explained

### Python-first Approach
The SDK emphasizes using Python's natural constructs (functions, classes, control flow) to build and manage agents. This reduces the learning curve and makes integration into existing Python projects more seamless.

### Function Tools
This is one of the most powerful features. You can make almost any Python function available to your agent as a "tool."
*   **Automatic Schema Generation**: The SDK can inspect your Python function's signature (including type hints) and automatically generate the necessary JSON schema that OpenAI models require for function calling.
*   **Pydantic Validation**: If you use Pydantic models for your function parameters, the SDK leverages Pydantic for robust input validation before your function is even called.

Example (conceptual):
```python
from agents import Agent, tool
from pydantic import BaseModel

class SearchQuery(BaseModel):
    keyword: str
    database: str = "case_law"

@tool
def search_legal_database(query: SearchQuery) -> list:
    """Searches the specified legal database for the given keyword."""
    # ... actual search logic ...
    print(f"Searching {query.database} for {query.keyword}")
    return [f"Result for {query.keyword} in {query.database}"]

# This tool can now be made available to an Agent.
```

### Handoffs
For complex workflows, you might need multiple agents with different specializations to collaborate. The "handoffs" feature allows one agent to delegate a task or a sub-task to another agent. This promotes modularity and allows you to build sophisticated multi-agent systems.

### Guardrails
Guardrails are checks or validations that can run in parallel with your agent's operations. If a guardrail condition is met (e.g., invalid input, policy violation), the agent's execution can be interrupted early. This is crucial for safety, compliance, and ensuring robust behavior.

### Tracing
The SDK includes built-in tracing capabilities. This means it can record the steps an agent takes, the tools it calls, the data exchanged, and the LLM interactions. Tracing is invaluable for:
*   **Debugging**: Understanding why an agent behaved in a certain way.
*   **Monitoring**: Observing agent performance in real-time.
*   **Evaluation**: Analyzing agent behavior for improvement and fine-tuning.
*   **Visualization**: Getting a clear picture of the agent's workflow.

## 5. Getting Started: Hello World

Here's the basic "Hello World" example provided by the SDK documentation:

```python
from agents import Agent, Runner

# 1. Define your Agent
# The 'instructions' parameter sets the system prompt for the agent.
agent = Agent(name="Assistant", instructions="You are a helpful assistant")

# 2. Use the Runner to execute the Agent with a prompt
# Runner.run_sync executes the agent synchronously.
result = Runner.run_sync(agent, "Briefly, what are the basic elements of a valid contract?")

# 3. Access the final output
# The 'result' object contains various details about the run, including the final output.
print(result.final_output)

# Expected Output (will vary based on the model, but conceptually):
# The basic elements of a valid contract generally include:
# 1. Offer and Acceptance
# 2. Consideration
# 3. Intention to create legal relations
# 4. Capacity of the parties
# 5. Legality of the subject matter.
```
Remember to have your `OPENAI_API_KEY` environment variable set when running this code.

## 6. Exploring Examples (with links to GitHub)

The OpenAI Agents SDK GitHub repository provides a rich set of examples to help you understand its capabilities. You can find them here: [OpenAI Agents Python Examples on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples)

Here's a summary of the example categories:

*   **`agent_patterns`**: Illustrates common agent design patterns.
    *   *Deterministic workflows*
    *   *Agents as tools*
    *   *Parallel agent execution*
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/agent_patterns)

*   **`basic`**: Showcases foundational SDK capabilities.
    *   *Dynamic system prompts*
    *   *Streaming outputs*
    *   *Lifecycle events*
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/basic)

*   **`tool_examples`**: Learn how to implement and integrate tools.
    *   *OAI hosted tools (e.g., web search, file search)*
    *   *Custom Python function tools*
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/tools)

*   **`model_providers`**: Explore how to use non-OpenAI models with the SDK.
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/model_providers)

*   **`handoffs`**: Practical examples of agent-to-agent handoffs.
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/handoffs)

*   **`mcp`**: Learn how to build agents compatible with MCP (Model Context Protocol).
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/mcp)

*   **`customer_service` and `research_bot`**: More built-out examples illustrating real-world applications.
    *   *`customer_service`*: Example customer service system for an airline.
    *   *`research_bot`*: A simple deep research agent.
    *   [View `research_bot` on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/research_bot)

*   **`voice`**: Examples of voice-enabled agents using OpenAI's TTS (Text-to-Speech) and STT (Speech-to-Text) models.
    *   [View on GitHub](https://github.com/openai/openai-agents-python/tree/main/examples/voice)

## 7. Important Notes & Best Practices

*   **`OPENAI_API_KEY`**: This is essential. Ensure it's correctly set in your environment for the SDK to communicate with OpenAI models.
*   **Start Simple**: Begin with basic agent definitions and gradually introduce tools and more complex patterns like handoffs.
*   **Clear Instructions**: The `instructions` (system prompt) for your agent are crucial. Be clear, concise, and specific about the agent's role, capabilities, and limitations.
*   **Tool Design**: When creating function tools:
    *   Keep them focused on a single, well-defined task.
    *   Use clear docstrings, as these can be used by the LLM to understand what the tool does.
    *   Leverage type hints and Pydantic models for robust validation.
*   **Iterate and Test**: Agent development is often an iterative process. Test your agents thoroughly with various inputs and scenarios. Use the tracing capabilities to debug and refine behavior.
*   **Cost Management**: Be mindful of API call costs, especially with complex agents that might make multiple LLM calls or use several tools in a single run.

This manual provides a starting point for understanding and using the OpenAI Agents SDK. For the most detailed and up-to-date information, always refer to the [official OpenAI Agents SDK documentation](https://openai.github.io/openai-agents-python/).
