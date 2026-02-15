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

