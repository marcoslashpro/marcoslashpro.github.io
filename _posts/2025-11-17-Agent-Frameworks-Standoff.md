---
layout: post
permalink: agent_frameworks_standoff
---


In 2025, AI is the most rapidly evolving field that there is. Everyone is jumping on the LLM-driven train, whether that
be for automations, tutoring, so on and so forth.

LLMs for themselves can be very smart, but at time useless. They do not possess up-to-date information on world
scenarios, they cannot test their thinking, they cannot take real actions.

An Agent is a solution to this problem, providing a way to use the natural language abilities of a LLM inside of an
environment that allows the model to take real actions, such as testing code snippets, executing web researches, and so
on.

The choice of framework when building an agent is an important one, and as of right now the number of agentic frameworks
that exist seems to overtake the number of models that are out there.

In this small research, we’ll create one simple multi-agent system with different Python agentic frameworks, trying to
understand the differences between each one, which one to use and why.

# The System

> This system is not a final production solution. It is a minimal prototype designed for comparative evaluation.
>

The **AI model** we’ll use is `claude-3-5-haiku-latest` hosted by **Anthropic**. While this choice is subjective, it
provides a stable and reliable LLM for the scope of this experiment.

The system design is illustrated below:

![image.png](/assets/images/agent_framework_standoff/framework_image.png)

**Figure 1**: A **DAG** representation of the multi-agent system.

---

## System Function

The system allows a user to interact through a **Natural Language User Interface (NLUI)**.

The **conversational agent** serves as the main interface. Its responsibilities are to:

1. Communicate directly with the user.
2. Delegate specific tasks to specialized **sub-agents**.
3. Integrate the outputs of sub-agents into its reasoning to provide the most accurate and complete response.

Sub-agents execute their assigned tasks and return results to the main agent, effectively enhancing its problem-solving
capabilities.

---

## System Structure

The system comprises **three ReAct-style agents**, each with a specialized role:

### Convo Agent

- Interface for user interaction.

- Handles natural language input and output.

- Delegates specialized tasks to sub-agents as needed.

### Math Agent

- Specializes in **mathematical reasoning** and problem-solving.

- **Tools:** A Python execution environment for testing calculations and executing code snippets, supporting libraries
like `math`.

### Research Agent

- Specializes in **online research** and information synthesis.

- **Tools:** Web-search functionality to query and collect up-to-date information from online sources.

---

This setup provides a clear, modular foundation to compare how different frameworks implement agent orchestration, tool
integration, and multi-agent coordination.

## Evaluation Methodology

To compare these frameworks fairly, we focus on **practical implementation, abstraction clarity, and reliability**,
rather than just theoretical feature lists. Each framework is evaluated by building a **simple multi-agent system** that
includes a conversational agent, a research agent, and a mathematics agent capable of reasoning and code execution.

The evaluation criteria include:

1. **Ease of Setup** – How quickly can a working system be built, including installation, model selection, and project
scaffolding?
2. **Abstraction Design** – How clear, consistent, and flexible are the core abstractions (`Agent`, `Tool`, `Task`,
etc.)?
3. **Workflow Orchestration** – How are multi-agent interactions handled? Does the framework provide declarative,
imperative, or hybrid orchestration?
4. **Extensibility** – How easy is it to integrate external tools, custom models, or new functionality?
5. **Testing & Evaluation** – Are there built-in mechanisms to validate outputs, handle retries, or mock models for safe
experimentation?
6. **Specialized Features** – Support for RAG workflows, code execution, multi-modality, or other domain-specific needs.

This methodology allows us to compare **real-world usability, maintainability, and robustness** across the frameworks,
while highlighting the trade-offs between **flexibility, safety, and ease-of-use**.

# LangChain

`langchain` is a comprehensive framework for building applications powered by large language models (LLMs). It provides
a rich set of base abstractions — such as models, embeddings, and vector stores — and makes it straightforward to create
custom implementations by extending just a few methods while the framework handles the rest.

Most common integrations (LLMs, embeddings, retrievers, vector stores, etc.) are already implemented, but the low-level
abstractions give developers a structured way to plug in custom logic where needed.

For multi-agent orchestration, `langchain` extends into companion packages such as `langgraph`, which is designed for
building complex graphs and workflows when low-level control is required. `langgraph` itself expands into sub-packages
like `langgraph_supervisor` and `langgraph_swarm`, both tailored to multi-agent systems.

The ecosystem is further supported by:

- **`langsmith`** → system activity monitoring and debugging

- **`langserve`** → deployment of agentic solutions to the cloud

`langchain` and `langgraph` complement each other: installing one typically installs the other, and some abstractions
live in one package while others in the other. To simplify, throughout this article I’ll use `langchain` to refer to
both, unless a distinction is important.

---

## Hello, World!

Let’s begin by setting up a working directory and installing the required packages:

```bash
mkdir langchainagent && cd langchainagent
uv add langchain langchain-community langchain-anthropic \
langgraph dotenv crewai-tools ddgs
```

- `langchain` and `langgraph` are the core packages

- `langchain-anthropic` adds access to Anthropic-hosted models

- `crewai-tools` provides useful ready-made tools (like a code interpreter)

Next, create a `__main__.py` file and an `agents.py` file.

We’ll start with a very simple chatbot agent that uses an Anthropic model, with no tools attached.

---

### Initializing the Model

There are two ways to set up a base model in `langchain`:

1. **Provider-agnostic** using `init_chat_model`
2. **Provider-specific** using classes from the respective `langchain_{provider}` package

```python
from langchain.chat_models import init_chat_model
from langchain_anthropic import ChatAnthropic
from dotenv import load_dotenv
from os import environ

load_dotenv()

# Provider-agnostic
llm = init_chat_model(
'anthropic:claude-3-5-haiku-latest', # syntax: {provider}:{model-name}
api_key=environ.get("ANTHROPIC_API_KEY"),
)

# Provider-specific
llm = ChatAnthropic(
model_name='claude-3-5-haiku-latest',
api_key=environ.get("ANTHROPIC_API_KEY"),
)

```

Both approaches are equivalent: under the hood, `init_chat_model` dispatches to the correct provider class (in this
case, `ChatAnthropic`). When using `init_chat_model`, you can pass the same parameters you would pass to the
provider-specific class.

---

### Creating the First Agent

In `langchain`, agents are essentially **directed acyclic graphs (DAGs)** of reasoning and tool calls. For common agent
patterns, `langgraph` provides pre-built implementations, such as the classic **ReAct agent**.

```python
from langgraph.prebuilt import create_react_agent

chatbot = create_react_agent(
model=llm,
prompt="You are Honest, Helpful, and Harmless.",
tools=[],
)
```

This creates a chatbot agent that can reason through tasks and use tools (if provided). With an empty toolset, it simply
answers directly.

---

### Running the Agent

Here’s a minimal `__main__.py` that takes user input and runs it through the agent:

```python
from argparse import ArgumentParser
from agents import chatbot

parser = ArgumentParser()
parser.add_argument("task")

def main():
args = parser.parse_args()
res = chatbot.invoke(
{"messages": [{"role": "user", "content": args.task}]},
)
print(res['messages'][-1].content)

if __name__ == "__main__":
main()
```

You can run this from the command line, passing in a query, and see the model’s reply.

---

## Tools

Tools extend agents with external capabilities. In `langchain`, tools can be implemented in multiple ways — here we’ll
use the `@tool` decorator, which turns a regular Python function (with type hints and docstrings) into a usable tool.

For this project, we’ll use two tools:

- `DuckDuckGoSearchRun` from `langchain_community.tools` for web research

- `CodeInterpreterTool` from `crewai-tools` for code execution (wrapped for compatibility)

```python
from crewai_tools import CodeInterpreterTool
from langchain_community.tools import DuckDuckGoSearchRun
from langchain.tools import tool

@tool(parse_docstring=True, infer_schema=True)
def code_execution_tool(code: str, libraries_used: list[str]) -> str:
"""
Tool for executing code.

Args:
code (str): Code to execute.
libraries_used (list[str]): Libraries required to run the code.

Returns:
str: Output of the execution, or an error message.
"""
try:
return CodeInterpreterTool().run_code_safety(code, libraries_used)
except Exception as e:
return f"Execution error: {str(e)}"
```

This wrapper ensures the tool conforms to `langchain`’s schema expectations while leveraging `crewai` under the hood.

---

## Multi-Agent Delegation

`langchain` supports multi-agent systems via:

- **`langgraph.Graph`** for custom workflows

- **`langgraph_supervisor`** for hierarchical “manager → worker” setups

- **`langgraph_swarm`** for autonomous swarms of agents

At the time of writing, I found `create_supervisor` unreliable, even when following the docs. As an alternative, I
implemented a delegation approach similar to Pydantic-AI: one “manager” agent coordinates specialist sub-agents via tool
calls.

---

### Specialist Agents

```python
researcher = create_react_agent(
llm,
name="researcher",
tools=[DuckDuckGoSearchRun()],
prompt=(
"You are a research specialist. You extract insights from search results, "
"filter out noise, and deliver clear, structured summaries with sources."
),
)

mathematician = create_react_agent(
model=init_chat_model(
"anthropic:claude-3-5-haiku-latest",
temperature=0.2,
api_key=environ.get("ANTHROPIC_API_KEY"),
),
name="mathematician",
tools=[code_execution_tool],
prompt=(
"You are a top mathematician. You break down problems step by step, "
"test solutions rigorously, and explain concepts simply. "
"Always provide your reasoning and final solution clearly."
),
)
```

---

### Wrapping Sub-Agents as Tools

We expose these specialists as tools so the manager can delegate to them:

```python
from langchain_core.messages import HumanMessage

@tool(parse_docstring=True, infer_schema=True)
def call_math_expert(problem: str) -> str:
"""Solve a mathematical problem using the math agent."""
try:
return mathematician.invoke(
{"messages": [HumanMessage(content=problem)]}
)["messages"][-1].content
except Exception as e:
return f"Math expert error: {str(e)}"

@tool(parse_docstring=True, infer_schema=True)
def call_research_expert(topic: str) -> str:
"""Research a topic using the research agent."""
try:
return researcher.invoke(
{"messages": [HumanMessage(content=topic)]}
)["messages"][-1].content
except Exception as e:
return f"Research expert error: {str(e)}"
```

---

### Manager Agent

Finally, we create a manager agent that can either answer directly or delegate via tools:

```python
chatbot = create_react_agent(
model=llm,
tools=[call_math_expert, call_research_expert],
prompt=(
"You are Honest, Helpful, and Harmless. "
"Your role is to coordinate the crew: analyze the user’s request, "
"decide whether to answer directly or delegate to specialists, "
"and deliver a final clear response."
),
)
```

This agent can now orchestrate tasks by routing them to the right specialist.

---

## Example Runs

```python
$ uv run langchainagent/ 'Relevant information on latest AI development trends'

>> The research provided a comprehensive overview of the latest AI development trends. The landscape is dynamic and
rapidly evolving, with significant advancements across multiple domains.

Key highlights include:
1. The continued evolution of generative AI with more sophisticated language models
2. A growing emphasis on ethical AI and responsible development
3. Specialized AI applications targeting specific industry needs
...
```

```python
$ uv run langchainagent/ 'Compute all primes ≤ 2,000,000 using a sieve.

Report:

prime_count = π(2_000_000)

prime_sum = sum of all primes ≤ 2_000_000

prime_head_hash = SHA256 of the first 100 primes joined by commas (no spaces).

Include time complexity analysis as code comments (not prose).'

>> Let me break down the key results for you:

1. Prime Count (π(2,000,000)): 148,933 primes
2. Prime Sum: 142,913,828,922
3. SHA256 Hash of First 100 Primes: 36c584fb3679dbafff494ae8c97696d76645a0df43f8eb9abb35c44a319c8ff3

The implementation uses the classic Sieve of Eratosthenes algorithm with time complexity O(n log log n), which is very
efficient for finding primes in a given range. The code includes detailed comments explaining the time and space
complexity as requested.

The algorithm works by...
```

---

## Final Thoughts

`langchain` provides a powerful toolkit for LLM applications. Its abstraction layers make it easy to extend and
customize workflows, and it’s particularly strong for building complex RAG pipelines.

That said, for **multi-agent systems**, the ecosystem feels less stable: the documentation is sparse, some official
solutions don’t work out of the box, and projects often require many additional dependencies. Community support is
strong, but this also means many solutions are fragile or poorly maintained.

For general LLM application development, `langchain` is a solid choice. For agentic or multi-agent systems, you may want to explore alternative frameworks — we’ll look at some of those later

# CrewAI

If we were to make a programming analogy, `crewAI` feels a lot like the **declarative paradigm**: you don’t specify
*how* something should happen, only *what* you want to achieve. The system handles the orchestration.

This makes `crewAI` stand out for its **ease of use**. The learning curve isn’t steep, and instead of spending time
wiring control flow, you focus on defining the core building blocks: **agents**, **tasks**, and **tools**. The
orchestration layer — the *crew* — ties them together.

The result is a framework where you describe intentions, and the execution plan emerges automatically.

---

### Getting Started

Setup is straightforward. Let’s create a project and install the dependencies:

```bash
mkdir agentcrew && cd agentcrew && uv init .
uv add crewai crewai-tools langchain-community duckduckgo-search
```

We install `crewai` and `crewai-tools` directly, plus `langchain-community` to import some existing tools, and
`duckduckgo-search` to support web queries.

By default, `crewAI` will generate a project scaffold for you. Personally, I prefer a lightweight structure:

```
.
├── crew.py
├── __main__.py
└── tools.py
```

This is all we need.

---

### Tools

Unlike `pydantic-ai`, `crewAI` doesn’t ship with built-in tools. Instead, it provides the scaffolding to build your own
— or you can install `crewai-tools` for batteries-included functionality. Even better, you can wrap existing `langchain`
tools for compatibility.

Example: creating a search tool with DuckDuckGo:

```python
from langchain_community.tools import DuckDuckGoSearchRun
from crewai.tools import BaseTool
from crewai_tools import CodeInterpreterTool
from pydantic import Field

class SearchTool(BaseTool):
name: str = "Search"
description: str = "Useful for search-based queries. Use this to find current information about markets, companies, and
trends."
search: DuckDuckGoSearchRun = Field(default_factory=DuckDuckGoSearchRun)

def _run(self, query: str) -> str:
"""Execute the search query and return results"""
try:
return self.search.run(query)
except Exception as e:
return f"Error performing search: {str(e)}"
```

This is simply a thin wrapper over a `langchain` tool. `name` and `description` make it discoverable to the model, and
`pydantic` ensures schema validation. Alongside this, we’ll use the built-in `CodeInterpreterTool`.

---

### Agents**

In `crewAI`, an agent is defined declaratively: you provide a **role**, a **goal**, and a **backstory**. The framework
takes care of message passing, memory, and orchestration.

Agents can be configured in YAML or in code; here we’ll stick with code for clarity:

```python
from crewai import Agent
from tools import SearchTool, CodeInterpreterTool

llm = "anthropic/claude-3-5-haiku-20241022"

chatbot = Agent(
role="Chatbot",
goal="Understand the user query and orchestrate the best plan to solve it.",
backstory=(
"You are Honest, Helpful, Harmless. "
"You analyze user requests, decide when to delegate, "
"and integrate outputs into clear responses."
),
llm=llm,
verbose=True,
reasoning=True
)

researcher = Agent(
role="Researcher",
goal="Find and synthesize comprehensive and up-to-date information about the given topic.",
backstory="You are an experienced research specialist, skilled at filtering noise and summarizing findings with
sources.",
llm=llm,
verbose=True,
tools=[SearchTool()]
)

mathematician = Agent(
role="Mathematician",
goal="Solve mathematical problems rigorously with step-by-step reasoning and code verification.",
backstory=(
"As one of the leading minds in mathematics, you break down complex problems, "
"test solutions with code, and explain clearly. "
"When using the CodeInterpreterTool, you must always declare the Python libraries you rely on."
),
llm=llm,
verbose=True,
allow_code_execution=True,
code_execution_mode="safe",
max_reasoning_attempts=3,
max_iter=10,
tools=[CodeInterpreterTool()]
)
```

Here you can see both the simplicity (`role`, `goal`, `backstory`) and the flexibility (optional parameters for
reasoning control, code execution modes, iteration limits).

---

### Tasks**

Tasks define *what the crew is supposed to accomplish*. They can be listed sequentially or managed hierarchically by a
manager agent.

```python
from crewai import Task

task = Task(
description=(
"The crew’s mission is to fulfill the user’s request ({query}).\n\n"
"- If the query is conversational, respond warmly.\n"
"- If it requires research, provide structured, up-to-date information.\n"
"- If it involves math, reason step-by-step and verify with code.\n\n"
"The chatbot should coordinate delegation and only output verified information."
),
expected_output="A polished, final response ready to present to the user."
)
```

The `{query}` parameter will be injected at runtime.

---

### Crew

At the top level, a **crew** is the orchestrator. You provide it with agents, tasks, and (optionally) a manager agent if
you want hierarchical execution.

```python
from crewai import Crew, Process

crew = Crew(
agents=[researcher, mathematician],
tasks=[task],
process=Process.hierarchical,
manager_agent=chatbot,
verbose=True
)

```

Here, the `chatbot` serves as the manager, directing the researcher and mathematician depending on the query.

---

### Running the Crew

We can expose this via a simple CLI in `__main__.py`:

```python
import argparse
from crew import crew

def main():
parser = argparse.ArgumentParser()
parser.add_argument("task")
args = parser.parse_args()

result = crew.kickoff(inputs={"query": args.task})
print(result)

if __name__ == "__main__":
main()
```

Notice how simple `kickoff` is: you pass the required inputs, and the crew executes the plan behind the scenes.

---

### Examples

Research:

```bash
$ uv run agentcrew "Show me some recent trends in AI development"

>> AI Development Trends 2023–2024
- Generative AI breakthroughs (GPT-4, Claude 3)
- Progress in multimodal reasoning
- Advances in robotics integration
...
```

Math:

```bash
$ uv run agentcrew "A train travels 120 miles in 2 hours. \
Then 150 miles in 3 hours. What is the average speed?"

>> Total distance: 270 miles
Total time: 5 hours
Average speed = 54 mph
```

---

### Final Thoughts

`crewAI` positions itself as one of the most accessible multi-agent frameworks available today. The declarative style
lowers the barrier to entry: you describe *what* you want, not *how* to coordinate agents. The abstractions are clear,
the documentation polished, and the ergonomics excellent.

There is, of course, a tradeoff: declarative abstractions can hide complexity, which may limit fine-grained control in
some cases. But the framework also exposes a lower-level **Flow** API — reminiscent of `langgraph` — for those who need
custom orchestration.

One notable gap, however, is the **lack of a native testing and evaluation layer**. Unlike `pydantic-ai`, which
integrates seamlessly with `pydantic-evals` for model output validation and rubric-based judgments, `crewAI` currently
leaves testing to the developer. This isn’t a dealbreaker, but it does mean you’ll need to bring your own testing
strategy if you want to validate agent outputs rigorously.

Overall, the balance between simplicity and power is spot on. `crewAI` makes multi-agent systems approachable without
sacrificing too much flexibility, and with a stronger story around evaluation, it could become one of the go-to
frameworks in this space.

# PydanticAI

You can think of `pydantic-ai` as a natural evolution of frameworks like `crewai`. Where other frameworks introduce a
wide array of primitives, `pydantic-ai` pares things down to a smaller set of well-chosen abstractions, with stronger
type safety, saner defaults, and fine-grained customization when you need it.

It guarantees structured inputs and outputs through `pydantic` models. It supports multiple model providers (or even
custom models), and its ergonomics feel very much like `FastAPI`: once you use both, the similarity is clear.

Workflow orchestration is not an afterthought. Built-in abstractions like `ModelRetry`, `FallbackModel`, and
`UnexpectedModelBehavior` make it straightforward to control how your system responds to errors, unexpected model
outputs, or degraded performance. You can express retries, failovers, and fallback logic without re-inventing
infrastructure.

Where `pydantic-ai` really shines is in **testing and evaluation**. By installing the companion `pydantic-evals`
package, you can write structured test cases and datasets, then run evaluations directly against your agents. Features
like `agent.override` allow you to swap out almost any parameter of an agent in tests, and `TestModel` makes mocking
easy without relying on live API calls. Testing is a neglected aspect of most agent frameworks, but here it’s treated as
a first-class capability.

The one notable gap is **RAG**. There’s no built-in support for vector stores or embeddings. You can pass in a retriever
tool from outside — for example, by leaning on `langchain` or `llamaindex` — but the responsibility lies outside the
framework. The docs mention that support is coming, and it will be interesting to see how they merge these two worlds.

With that context in mind, let’s see how it works in practice.

---

### Hello World

We’ll start with a minimal setup.

`pydantic-ai` offers a slim installation variant where you only pull in the providers you need. Here we’ll install the
Anthropic provider:

```bash
mkdir pydanticagent && cd pydanticagent && uv init .
uv add pydantic-ai-slim[anthropic] dotenv
```

Now we can build a model. Models and providers are separate abstractions — here we’ll use `AnthropicModel` with an
`AnthropicProvider`:

```python
from pydantic_ai.models.anthropic import AnthropicModel
from pydantic_ai.providers.anthropic import AnthropicProvider

from dotenv import load_dotenv
from os import environ

load_dotenv()

model = AnthropicModel(
model_name='claude-3-5-haiku-latest',
provider=AnthropicProvider(api_key=environ.get("ANTHROPIC_API_KEY"))
)
```

With the model ready, creating an agent is trivial:

```python
from pydantic_ai import Agent

agent = Agent(
model=model,
instructions="You are Honest, Helpful and Harmless"
)

agent.run_sync('Can you just say "Hello World!"?')

>> Hello World!
```

That’s our first working agent. Let’s scale it up.

---

### Agents and Tools

We’ll define three agents:

- A **chatbot** coordinator,

- A **researcher** with web search capabilities,

- And a **mathematician** with code execution.

Conveniently, `pydantic-ai` ships with exactly these two built-in tools (`WebSearchTool`, `CodeExecutionTool`).

```python
from pydantic_ai import WebSearchTool, CodeExecutionTool

chatbot = Agent(
model=model,
instructions=(
"You are Honest, Helpful, Harmless. "
"Your main job is to coordinate the crew: analyze the user's request, "
"decide whether to answer directly or delegate to specialists, and "
"integrate their outputs into a clear response."
),
name='Chatbot'
)

researcher = Agent(
model=model,
instructions=(
"You are an experienced research specialist. "
"You excel at extracting key insights from search results, filtering noise, "
"and delivering structured summaries with sources."
),
builtin_tools=[WebSearchTool()]
)

mathematician = Agent(
model=model,
instructions=(
"You are one of the leading minds in mathematics. "
"You break problems into steps, verify results, "
"and explain concepts clearly, even to a child."
),
builtin_tools=[CodeExecutionTool()]
)
```

---

### Delegation with Tools

The recommended way to enable multi-agent handoff is by exposing specialist agents as tools for the manager agent.

Here we wrap the researcher and mathematician into tools for the chatbot:

```python
@chatbot.tool_plain(docstring_format='google')
async def call_research_agent(topic: str) -> str:
"""
Helpful tool that calls a specialist research agent.
Use this anytime you need to search the web.
"""
res = await researcher.run(topic)
return res.output

@chatbot.tool_plain(docstring_format='google')
async def call_math_agent(problem: str) -> str:
"""
Helpful tool that calls a math expert agent.
Use this tool for solving complex math problems.
"""
res = await mathematician.run(problem)
return res.output
```

The framework parses function docstrings and type hints to automatically infer schemas. This makes agent handoff smooth,
and the similarity to `FastAPI`’s dependency injection system is no accident.

---

### Running the System

We can wrap this in a simple CLI:

```python
from argparse import ArgumentParser

parser = ArgumentParser()
parser.add_argument('task')

def main():
args = parser.parse_args()
res = chatbot.run_sync(args.task)
print(res.output)

if __name__ == "__main__":
main()
```

---

### Examples

Research:

```bash
$ uv run pydanticagent/ "Latest news on AI development"

>> Let me summarize the key highlights of recent AI developments:

- AlphaGeometry: An AI system solving Olympiad-level geometry
- Progress in protein simulation and brain mapping
...
```

Mathematics:

```bash
$ uv run pydanticagent/ "A train travels 120 miles in 2 hours. \
Then it slows down and travels 150 miles in 3 hours. \
What was the train’s average speed for the entire trip?"

>> Let me break down the solution for you:
1. First segment: 120 miles in 2 hours
2. Second segment: 150 miles in 3 hours
3. Total distance: 270 miles
4. Total time: 5 hours
5. Average speed = 54 mph
```

---

### Testing and Evaluation

Where most frameworks stop, `pydantic-ai` keeps going. Testing is not an afterthought — it’s part of the design.

By installing `pydantic-evals`, you get access to a testing ecosystem:

- `Case` for defining expected inputs,

- `Dataset` for grouping cases,

- Evaluators like `IsInstance` and `LLMJudge`,

- And support for mocking with `TestModel` or overrides via `agent.override`.

Here’s a simple evaluation for the mathematician agent:

```python
from pydantic_evals import Case, Dataset
from pydantic_evals.evaluators import LLMJudge, IsInstance

case1 = Case(
inputs="Compute all primes ≤ 2,000,000 using a sieve..."
)

dataset = Dataset(
cases=[case1],
evaluators=[
IsInstance(type_name="MathResponseModel"),
LLMJudge(
rubric="Response should contain step by step explanation.",
model=evaluator
)
]
)

report = dataset.evaluate_sync(solve_problem)
print(report)
```

Extending this to research tasks is just as easy. You can verify both the type of outputs and their quality against
rubrics, even using another model as a judge.

This raises a bigger question: could AI systems be tested as rigorously as APIs, with unit tests, integration tests, and
CI pipelines? `pydantic-ai` takes a significant step in that direction.

---

### Final Thoughts

I came away impressed by how much `pydantic-ai` gets right. The learning curve isn’t steep, yet the framework supports
deep customization. Strong type safety, built-in control-flow mechanisms (`ModelRetry`, `FallbackModel`,
`UnexpectedModelBehavior`), and first-class testing make it stand out.

The lack of built-in RAG is a limitation, but the modularity of the design makes it feasible to bridge with other
frameworks. Combined with `pydantic-evals` and `TestModel`, you get not just agents but a testable, reliable platform.

This feels like a mature, stable, and well-engineered library. Highly recommended.

# Comparative Wrap-Up

Having explored **LangChain**, **Pydantic-AI**, and **CrewAI** in depth, it becomes clear that these frameworks don’t
just differ in their abstractions and workflows — they almost embody **different programming paradigms applied to
agentic systems**.

- **LangChain → Imperative Programming**

You tell the framework *exactly how* to connect things, step by step. It shines in flexibility and its vast ecosystem of
integrations (`VectorStores`, `Embeddings`, `RAG`, toolkits, memory modules, etc.). But that power comes at a cost: the
learning curve is steep, the abstractions are often leaky, and testing is not a first-class citizen. LangChain is
perfect when you need total control and are willing to accept complexity to get it.

- **CrewAI → Declarative Programming**

You don’t describe *how* the workflow should run, only *what* needs to be achieved. Agents, tasks, and tools are defined
with simple goals, roles, and backstories, while orchestration is handled by the framework. It is by far the easiest to
set up and get running — the “happy path” is incredibly smooth. But its strength is also its weakness: less fine-grained
control, and critically, **a lack of robust testing utilities**, which raises concerns for production reliability.
CrewAI is the fastest way to prototype, but more fragile when scaling.

- **Pydantic-AI → Strongly Typed Functional Programming**

Here, correctness and safety are front and center. Every input, output, and intermediate step is backed by `pydantic`
models. Evaluation (`pydantic-evals`), control flow abstractions (`ModelRetry`, `FallbackModel`,
`UnexpectedModelBehavior`), and test models make it arguably the **most reliable and testable** framework of the three.
The tradeoff is that it currently lacks built-in RAG support, forcing you to integrate with other ecosystems (LangChain,
LlamaIndex) for retrieval-heavy applications. Still, it feels like the most “engineered” approach, with modularity and
type safety ensuring long-term maintainability.


---

# Final Thoughts

Each framework excels in a different dimension:

- **LangChain**: unmatched breadth of integrations and workflow control, but often unwieldy.

- **CrewAI**: smooth declarative abstractions and rapid prototyping, but limited testing and weaker guarantees.

- **Pydantic-AI**: type safety, modular design, and evaluation built in, but missing native RAG and still maturing.

In many ways, these trade-offs mirror broader software engineering lessons: **flexibility vs. safety vs. ease of use**.
None of these frameworks is objectively “better” — the right choice depends on your project’s needs:

- If you want **maximum control** → pick **LangChain**.

- If you want **ease of use and speed** → pick **CrewAI**.

- If you want **robustness and testability** → pick **Pydantic-AI**.

This standoff shows that while we are still early in the evolution of agentic frameworks, we already see clear
philosophical directions emerging. The future might well involve combining strengths: the **breadth of LangChain**, the
**declarative simplicity of CrewAI**, and the **safety of Pydantic-AI**. Until then, the framework you choose says as
much about your **engineering philosophy** as it does about your product requirements.

END

* TOC
{:toc}
