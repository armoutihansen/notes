---
layer: 05_ai_engineering
type: ai_system
status: growing
tags: [multi-agent, crewai, autogpt, orchestrator, agents, llm]
created: 2026-03-05
---

# Multi-Agent Systems

## Goal

Coordinate multiple specialized LLM agents to solve complex tasks through collaboration and division of labor. A single LLM call has bounded context, compute, and reliability. Multi-agent architectures overcome these limits by decomposing problems into parallel workstreams, applying specialized agents to subproblems, and implementing error-correction through critique-and-revision patterns. This enables tackling tasks that exceed a single agent's scope: large-scale research, software development pipelines, automated content production, and long-horizon planning.

## Architecture

### Orchestrator-Worker Pattern
A supervisor (orchestrator) agent receives the high-level goal, decomposes it into subtasks, routes each subtask to the appropriate specialist agent, collects results, and synthesizes the final output. The orchestrator is typically a capable general LLM; worker agents may be smaller models specialized for their domain.

```
User Goal
    ↓
Orchestrator (GPT-4o)
  ├── Research Agent → web search, document retrieval
  ├── Analyst Agent → data processing, calculations
  ├── Writer Agent → drafting, summarization
  └── Reviewer Agent → quality check, fact verification
    ↓
Aggregated final output
```

### Sequential Pipelines (A → B → C)
Each agent's output becomes the input for the next. Suited to tasks with natural ordering: research → outline → draft → edit → review. Simple to implement, easy to debug. No parallelism, so total latency = sum of each agent's latency.

### Hierarchical (Manager → Worker layers)
The orchestrator delegates to team managers, each of whom orchestrates their own workers. Enables arbitrarily complex decomposition for enterprise-scale tasks. High coordination overhead; use only when task complexity justifies it.

### Debate / Critique Patterns
- **Generator-Critic**: One agent generates output; a separate critic agent evaluates it for errors, bias, or missing information. The generator revises based on critique. Improves output quality without human review.
- **Multi-agent debate**: Multiple agents independently produce answers; a final arbiter synthesizes or selects the best response. Reduces individual model errors.

### CrewAI Framework
CrewAI models agents as crew members with defined roles, goals, and backstories. Each agent has access to a set of tools. The `Crew` object orchestrates execution in sequential or hierarchical mode:
```python
from crewai import Agent, Task, Crew, Process

researcher = Agent(role="Research Specialist", goal="Find accurate information", backstory="...", tools=[search_tool])
writer = Agent(role="Technical Writer", goal="Produce clear documentation", backstory="...", tools=[])

research_task = Task(description="Research RAG architectures", agent=researcher)
writing_task = Task(description="Write a technical summary", agent=writer, context=[research_task])

crew = Crew(agents=[researcher, writer], tasks=[research_task, writing_task], process=Process.sequential)
result = crew.kickoff()
```

### AutoGPT / Autonomous Long-Horizon Agents
Fully autonomous agents that maintain long-term goals across many iterations, with persistent memory and the ability to spawn sub-agents. High autonomy; high risk of runaway cost and off-task behavior. Appropriate for offline/batch tasks with human review at checkpoints.

## Components

**Orchestrator**: Receives the user goal, maintains task decomposition state, routes subtasks, aggregates results, decides when the overall goal is satisfied. Typically the most capable model in the system.

**Specialist agents**: Researcher (retrieval, web search), Coder (code generation, execution), Reviewer (quality assurance, fact-checking), Planner (task decomposition), Writer (text generation). Each agent has a focused system prompt and a curated tool set.

**Shared memory**: 
- Conversation history passed between agents as context.
- Vector store for long-term knowledge accumulation during a run.
- A shared workspace (structured dict or document) that agents read/write to pass intermediate results.

**Tool registry**: Centralized registry of available tools, accessible to agents according to their roles. The orchestrator may dynamically assign tools to workers per-task.

**Human escalation layer**: Defined checkpoints at which the system pauses for human review — before irreversible actions, when confidence is low, or after a fixed number of autonomous steps. Critical for production deployments.

## Evaluation

**Task completion rate**: What fraction of end-to-end tasks are completed correctly without human intervention? The primary business metric.

**Step efficiency**: Number of LLM calls (and tool executions) required to complete the task. Lower is better; inefficient agents loop unnecessarily or make redundant tool calls.

**Consistency across agents**: When multiple agents produce overlapping outputs, do they agree? High disagreement rate signals context handoff problems or ambiguous task definitions.

**Escalation rate**: Proportion of tasks requiring human intervention. Target is low (high autonomy) but not zero (some tasks genuinely need human judgment).

**Cost per task**: Total LLM + tool execution cost averaged per successfully completed task. Important for business viability.

## Failure Modes

**Context handoff loss**: Agents receive insufficient context about prior steps and make decisions that conflict with or ignore earlier results. Mitigation: include full task history and prior agent outputs in each agent's context; use a shared workspace document.

**Error propagation through pipeline**: A mistake in an early stage (wrong research finding, incorrect calculation) propagates through all downstream stages, producing a confidently wrong final output. Mitigation: include a reviewing agent at each critical stage boundary.

**Infinite loops between agents**: Agent A delegates to Agent B, which delegates back to Agent A. Mitigation: strict task ownership in the orchestrator; track delegation depth and cap it.

**Hallucinated inter-agent communication**: An agent fabricates results from a tool or sub-agent call that it was supposed to execute but didn't. Mitigation: require all inter-agent handoffs to pass through the tool execution layer (not LLM-to-LLM free text); log and verify all tool calls.

**Cost blow-up with uncontrolled recursion**: Hierarchical or autonomous agents can spawn unbounded sub-agent chains. Mitigation: global step budget enforced by the orchestrator; cost alerts at runtime.

## Cost / Latency

Each agent invocation requires at least one LLM API call plus any tool executions. For a linear N-agent pipeline:

| Configuration | LLM Calls | Approx. Cost (GPT-4o) | Latency |
|---|---|---|---|
| Single agent (10 steps) | ~10 | ~$0.05–0.50 | ~5–15s |
| 3-agent sequential pipeline | ~15–30 | ~$0.10–1.00 | ~15–45s |
| Hierarchical (2 layers, 6 agents) | ~30–60 | ~$0.50–5.00 | ~30–120s |

**Optimization strategies**:
- Async / parallel execution for independent subtasks within a layer.
- Use smaller, specialized models (GPT-4o-mini, Llama 3 8B) for worker agents; reserve frontier models for the orchestrator.
- Cache repeated sub-computations (e.g., the same document retrieval across multiple agents in a run).
- Compress context passed between agents to essential information only.

## Links
- [[agentic_loop|Agentic Loop]]
- [[function_calling|Function Calling]]
- [[rag_architecture|RAG Architecture]]
- [[prompt_injection_and_guardrails|Prompt Injection and Guardrails]]
