# CrewAI-projects

Think of **CrewAI** as a movie set: you aren't just writing a script; you’re hiring actors, giving them backstories, and appointing a director to make sure they don't step on each other's lines.

Here is the "Mental Model" you need to master CrewAI, broken down from the core concepts to a working example.

---

## 🏗️ The Four Pillars of CrewAI

| Component | What it represents | The "Secret Sauce" |
| --- | --- | --- |
| **Agent** | The "Worker" | Defined by **Role**, **Goal**, and **Backstory**. The backstory is actually the system prompt that shapes their "personality." |
| **Task** | The "To-Do" | Defined by **Description** and **Expected Output**. High-quality "Expected Output" descriptions are the #1 way to stop AI hallucinations. |
| **Crew** | The "Organization" | This is where you bring Agents and Tasks together and decide the **Process** (how they talk to each other). |
| **Tools** | The "Equipment" | Functions or APIs (like Google Search, PDF readers, or Python interpreters) that agents can use to get facts. |

---

## 🔄 The "Workflow" (Process)

CrewAI stands out because of how agents collaborate. You have three main ways to run a crew:

1. **Sequential (Default):** Agent A finishes Task A  passes result to Agent B for Task B.
2. **Hierarchical:** You appoint a **Manager LLM**. The manager looks at the tasks, decides which agent does what, and reviews their work before finishing.
3. **Consensual:** Agents can collaborate and "vote" on a result (less common, more complex).

---

## 🛠️ Let's Build a "Research & Write" Crew

In modern CrewAI (using the `@CrewBase` pattern you started), we separate the **Logic** (Python) from the **Prompts** (YAML).

### Step 1: Define the "Brain" (`agents.yaml`)

```yaml
researcher:
  role: >
    Senior Technology Researcher
  goal: >
    Uncover groundbreaking developments in {topic}
  backstory: >
    You are an expert at spotting trends. You don't just find facts; 
    you find the "why" behind them.

writer:
  role: >
    Tech Content Strategist
  goal: >
    Craft compelling content based on research
  backstory: >
    You take complex data and turn it into stories that humans 
    actually want to read.

```

### Step 2: Define the "Instructions" (`tasks.yaml`)

```yaml
research_task:
  description: >
    Analyze the latest trends in {topic} for 2026.
  expected_output: >
    A list of 5 key trends with a brief explanation for each.

write_task:
  description: >
    Use the research provided to write a 3-paragraph blog post.
  expected_output: >
    A markdown formatted blog post ready for publication.

```

### Step 3: The Python Logic

```python
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task

@CrewBase
class ContentCrew():
    agents_config = 'config/agents.yaml'
    tasks_config = 'config/tasks.yaml'

    @agent
    def researcher(self) -> Agent:
        return Agent(config=self.agents_config['researcher'])

    @agent
    def writer(self) -> Agent:
        return Agent(config=self.agents_config['writer'])

    @task
    def research_task(self) -> Task:
        return Task(config=self.tasks_config['research_task'])

    @task
    def write_task(self) -> Task:
        return Task(config=self.tasks_config['write_task'])

    @crew
    def crew(self) -> Crew:
        return Crew(
            agents=self.agents, # Automatically picks up @agent methods
            tasks=self.tasks,   # Automatically picks up @task methods
            process=Process.sequential
        )

# To run it:
# result = ContentCrew().crew().kickoff(inputs={'topic': 'AI in Healthcare'})

```

---

## 💡 3 Golden Rules for CrewAI Success

1. **Be Explicit with Outputs:** Don't just say "Write a report." Say "Expected Output: A 5-page PDF with a table of contents and a summary section."
2. **Give Tools to the Right People:** Don't give a "Search Tool" to the Writer; give it to the Researcher. This keeps the Writer focused on style, not browsing.
3. **The "Manager" LLM:** If your crew has more than 3 agents, use `process=Process.hierarchical`. It prevents the agents from getting "lost" in the sequence.



----------------------------------------------



Great — this is a full **CrewAI Crew definition using the class-based pattern**.

Let’s break down exactly what this code is doing.

---

# 🔎 High-Level Overview

This defines a **Debate system** with:

* 2 agents:

  * `debater`
  * `judge`

* 3 tasks:

  * `propose`
  * `oppose`
  * `decide`

* A sequential execution flow

This is a structured multi-agent workflow.

---

# 📦 Imports

```python
from crewai import Agent, Crew, Process, Task
from crewai.project import CrewBase, agent, crew, task
```

Important pieces:

* `Agent` → defines an agent
* `Task` → defines work assigned to an agent
* `Crew` → orchestrates agents + tasks
* `Process` → defines execution style
* `CrewBase` → special decorator for class-based crew setup
* `@agent`, `@task`, `@crew` → decorators that auto-register components

---

# 🏗 The Class Structure

```python
@CrewBase
class Debate():
```

`@CrewBase` does something important:

It:

* Loads configuration files
* Automatically collects agents
* Automatically collects tasks
* Connects everything cleanly

This is a structured way to define a Crew project.

---

# 📁 Configuration Files

```python
agents_config = 'config/agents.yaml'
tasks_config = 'config/tasks.yaml'
```

These point to external YAML files.

So:

* Agent definitions live in `agents.yaml`
* Task definitions live in `tasks.yaml`

This keeps code clean and modular.

---

# 🤖 Agent Definitions

## Debater

```python
@agent
def debater(self) -> Agent:
    return Agent(
        config=self.agents_config['debater'],
        verbose=True
    )
```

What happens here:

* It loads the `debater` section from `agents.yaml`
* That config contains:

  * role
  * goal
  * backstory
  * llm

`@agent` decorator:

* Registers this as an agent
* Automatically adds it to `self.agents`

---

## Judge

Same structure:

```python
@agent
def judge(self) -> Agent:
```

Also auto-registered.

---

# 🧠 Task Definitions

You have three tasks:

```python
@task
def propose(self) -> Task:
```

```python
@task
def oppose(self) -> Task:
```

```python
@task
def decide(self) -> Task:
```

Each loads its configuration from `tasks.yaml`.

Important:
Tasks usually define:

* description
* expected_output
* which agent executes it

The `@task` decorator:

* Registers the task
* Automatically adds it to `self.tasks`

---

# 🔄 The Crew Orchestration

```python
@crew
def crew(self) -> Crew:
```

This builds the actual system.

```python
return Crew(
    agents=self.agents,
    tasks=self.tasks,
    process=Process.sequential,
    verbose=True,
)
```

### Key Points:

### ✅ `agents=self.agents`

This list is automatically created from all `@agent` methods.

### ✅ `tasks=self.tasks`

Auto-collected from all `@task` methods.

### ✅ `process=Process.sequential`

This is very important.

It means:

Tasks execute in order:

1. `propose`
2. `oppose`
3. `decide`

Each task runs after the previous one completes.

---

# 🧠 What This Debate Flow Likely Does

Step 1: `propose`

* Debater argues FOR the motion.

Step 2: `oppose`

* Debater (same agent) argues AGAINST the motion.

Step 3: `decide`

* Judge evaluates both arguments.
* Chooses winner.

Because it's sequential, the judge receives prior outputs.

---

# 🏗 Architectural Insight

This is a hybrid between:

* Agentic reasoning (multiple roles)
* Structured workflow (sequential process)

Even though this is CrewAI “Crews,”
it’s actually quite controlled because:

* The task order is fixed.
* The process is sequential.

---

# 🔥 Important Design Observation

You only have ONE debater agent.

So that means:

* The same agent likely writes both sides.

If you wanted a more realistic debate, you might create:

* `proponent`
* `opponent`
* `judge`

Three separate agents.

---

# 🧩 Why This Pattern Is Powerful

This class-based structure:

* Separates configuration from logic
* Keeps agents reusable
* Keeps tasks modular
* Makes scaling easier

It’s cleaner than defining everything inline.

---

# 🧠 Mental Model

Think of this file as:

"Assembly instructions for a debate simulation."

* Agents = workers
* Tasks = assignments
* Crew = project manager
* Process = workflow style

---

# ⚙ What Happens When You Run It?

Somewhere else in your project, you’ll have:

```python
debate = Debate()
result = debate.crew().kickoff(inputs={"motion": "AI should replace teachers"})
```

Then:

1. Agents load.
2. Tasks load.
3. Motion injected.
4. Tasks execute sequentially.
5. Judge returns final decision.

---

# 🎯 Summary

This code:

* Defines two agents (debater + judge)
* Defines three tasks (propose, oppose, decide)
* Runs them sequentially
* Uses external YAML configs
* Uses CrewAI’s structured class-based pattern

It creates a clean multi-agent debate pipeline.

---
