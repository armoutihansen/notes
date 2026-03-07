---
layer: 08_implementations
type: application
status: growing
tags: [pattern, reasoning, llm]
created: 2026-05-10
---

# Agentic Loop Pattern

## Purpose

The agentic loop is the fundamental execution pattern for LLM-based autonomous agents. An LLM iteratively observes the environment, reasons about the next action, calls a tool, receives the result, and continues until the goal is achieved. This note covers three implementation levels: a single-agent ReAct loop with LangChain, a multi-agent workflow with CrewAI, and an MCP (Model Context Protocol) tool-calling example.

### Examples

**Research agent**: Given a question, searches the web, reads sources, extracts key facts, writes a structured report — iterating until coverage is sufficient.

**Data analysis agent**: Given a CSV and a question, writes Python code, executes it in a sandbox, interprets the output, and refines if the result is incorrect.

---

## Architecture

```
User goal
    ↓
[Think: LLM reasons — what is current state? what tool to use?]
    ↓
[Act: call tool with structured arguments]
    ↓
[Observe: append tool result to context]
    ↓
[Evaluate: is goal achieved? yes → respond; no → loop]
```

---

## Setup

```bash
pip install langchain langchain-openai
pip install crewai crewai-tools
```

---

## Single-Agent ReAct Loop (LangChain)

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_react_agent, AgentExecutor
from langchain.tools import Tool
from langchain import hub

# --- Define tools ---
def search_web(query: str) -> str:
    """Search the web and return a summary of results."""
    # Replace with real search tool (SerpAPI, Tavily, etc.)
    return f"[Search results for '{query}': relevant info here...]"

def python_repl(code: str) -> str:
    """Execute Python code and return stdout output."""
    import io, contextlib
    output = io.StringIO()
    with contextlib.redirect_stdout(output):
        exec(code, {})
    return output.getvalue()

tools = [
    Tool(name="search_web",  func=search_web,  description="Search the internet for current information. Input: search query string."),
    Tool(name="python_repl", func=python_repl, description="Execute Python code. Input: valid Python code string. Returns stdout."),
]

# --- Agent setup (ReAct: Reason + Act) ---
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
prompt = hub.pull("hwchase17/react")   # standard ReAct prompt template

agent = create_react_agent(llm=llm, tools=tools, prompt=prompt)
agent_executor = AgentExecutor(
    agent=agent,
    tools=tools,
    verbose=True,
    max_iterations=10,
    handle_parsing_errors=True,
)

result = agent_executor.invoke({"input": "What is the current Python version and what new feature was added in the latest release?"})
print(result["output"])
```

---

## Tool-Calling Agent (Structured Outputs)

```python
from langchain_openai import ChatOpenAI
from langchain.agents import create_openai_tools_agent, AgentExecutor
from langchain_core.tools import tool

@tool
def get_weather(city: str) -> str:
    """Get current weather for a city. Returns temperature and conditions."""
    # Replace with real weather API call
    return f"Weather in {city}: 22°C, partly cloudy"

@tool
def calculate(expression: str) -> str:
    """Evaluate a mathematical expression safely. Returns the result."""
    try:
        return str(eval(expression, {"__builtins__": {}}, {}))
    except Exception as e:
        return f"Error: {e}"

tools = [get_weather, calculate]
llm   = ChatOpenAI(model="gpt-4o", temperature=0).bind_tools(tools)

from langchain import hub
prompt = hub.pull("hwchase17/openai-tools-agent")
agent  = create_openai_tools_agent(llm=llm, tools=tools, prompt=prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

result = executor.invoke({"input": "What is the weather in Paris, and what is 42 * 1.15?"})
print(result["output"])
```

---

## Multi-Agent Workflow (CrewAI)

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool

search_tool = SerperDevTool()  # web search (requires SERPER_API_KEY)

# --- Define specialist agents ---
researcher = Agent(
    role="Research Analyst",
    goal="Find accurate, up-to-date information on the assigned topic",
    backstory="Expert at synthesizing information from multiple sources",
    tools=[search_tool],
    verbose=True,
    llm=ChatOpenAI(model="gpt-4o-mini"),
)

writer = Agent(
    role="Technical Writer",
    goal="Transform research findings into clear, structured reports",
    backstory="Skilled at translating complex research into accessible prose",
    tools=[],
    verbose=True,
    llm=ChatOpenAI(model="gpt-4o-mini"),
)

# --- Define tasks ---
research_task = Task(
    description="Research the latest advancements in {topic}. Cover key developments from 2024, leading papers, and practical applications.",
    expected_output="A structured bullet-point summary with 5–8 key findings and sources.",
    agent=researcher,
)

writing_task = Task(
    description="Using the research summary provided, write a 3-paragraph executive overview of {topic} suitable for a technical audience.",
    expected_output="A 3-paragraph executive overview in markdown.",
    agent=writer,
    context=[research_task],   # writer receives researcher's output
)

# --- Assemble crew ---
crew = Crew(
    agents=[researcher, writer],
    tasks=[research_task, writing_task],
    process=Process.sequential,  # or Process.hierarchical for manager-managed
    verbose=True,
)

result = crew.kickoff(inputs={"topic": "multimodal large language models"})
print(result.raw)
```

---

## MCP Tool-Calling Example

```python
# MCP (Model Context Protocol) exposes tools as standardised JSON-RPC endpoints
# The LLM calls the MCP server; the server routes to the right tool handler

# Server side (FastAPI MCP server — see mcp_server_implementation.md)
# Client side: LLM calls tools via structured tool_call format

from openai import OpenAI

client = OpenAI()
tools_spec = [
    {
        "type": "function",
        "function": {
            "name": "read_file",
            "description": "Read a file from the project repository",
            "parameters": {
                "type": "object",
                "properties": {"path": {"type": "string", "description": "Relative file path"}},
                "required": ["path"],
            },
        },
    }
]

messages = [{"role": "user", "content": "Show me the contents of src/main.py"}]
response = client.chat.completions.create(model="gpt-4o", tools=tools_spec, messages=messages)

# Parse tool call and dispatch to MCP server
tool_call = response.choices[0].message.tool_calls[0]
print(f"Tool: {tool_call.function.name}, Args: {tool_call.function.arguments}")
```

---

## Agent Design Principles

| Concern | Recommendation |
|---|---|
| Max iterations | Set hard limit (10–20); prevents runaway loops |
| Tool errors | Return error string (not exception) — agent can retry with different args |
| Memory | Use `ConversationBufferWindowMemory` for recent context; vector store for long-term |
| Streaming | Use `astream_events` for real-time UI token streaming |
| Evaluation | Log every trace to LangSmith; measure task success rate and avg iterations |
| Safety | Wrap tool execution in try/except; validate inputs before exec (no arbitrary code exec in prod) |

---

## Links

**AI Engineering**
- [[06_ai_engineering/04_rag_and_agents/agentic_loop|Agentic Loop]] — ReAct, planning patterns, memory types theory
- [[06_ai_engineering/04_rag_and_agents/multi_agent_systems|Multi-Agent Systems]] — supervisor patterns, CrewAI, AutoGen architectures
- [[06_ai_engineering/04_rag_and_agents/function_calling|Function Calling]] — structured tool use and JSON schema tool specs

**System Patterns**
- [[mcp_server_implementation|MCP Server Implementation]] — building an MCP tool server for agents to call
- [[langsmith_llm_observability|LangSmith LLM Observability]] — trace agent runs for debugging

**End-to-End Examples**
- [[llm_coding_assistant|LLM Coding Assistant]] — multi-agent coding assistant using this pattern
