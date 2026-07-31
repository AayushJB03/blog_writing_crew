# BlogWritingCrew

A multi-agent AI system, built with [crewAI](https://crewai.com), that turns a single topic into a polished blog post. Three agents — a researcher, a writer, and an editor — work one after another, each handing its output to the next.

## How it was built

The project started from the standard `crewai create crew` template and was shaped into a 3-agent pipeline:

- **`config/agents.yaml`** — defines each agent's role, goal, and backstory (Researcher, Writer, Editor)
- **`config/tasks.yaml`** — defines what each agent has to do and what "done" looks like
- **`crew.py`** — wires agents to tasks and runs them as a `Process.sequential` crew
- **`main.py`** — the entry point `crewai run` calls, with a hardcoded `topic` input
- **LLM** — Google Gemini, configured entirely through `.env` (`MODEL=gemini/...` + `GEMINI_API_KEY`), no code changes needed

### Architecture

```mermaid
flowchart LR
    U["crewai run"] --> M["main.py: run()"]
    M --> C["BlogWritingCrew.crew()"]

    subgraph Crew["Sequential Process"]
        direction LR
        R["Researcher Agent"] -->|research brief| W["Writer Agent"]
        W -->|draft post| E["Editor Agent"]
    end

    C --> R
    E --> O["output/blog_post.md"]
```

### Task pipeline

```mermaid
sequenceDiagram
    participant Main as main.py
    participant Crew as BlogWritingCrew
    participant LLM as Gemini

    Main->>Crew: kickoff(inputs={topic, current_year})
    Crew->>LLM: research_task (Researcher)
    LLM-->>Crew: research brief
    Crew->>LLM: writing_task (Writer)
    LLM-->>Crew: draft blog post
    Crew->>LLM: editing_task (Editor)
    LLM-->>Crew: polished post
    Crew-->>Main: output/blog_post.md
```

### Project structure

```mermaid
flowchart TD
    root["blog_writing_crew/"] --> src["src/blog_writing_crew/"]
    src --> crewpy["crew.py — agents + tasks + crew definition"]
    src --> mainpy["main.py — run / train / replay / test entry points"]
    src --> config["config/"]
    config --> agentsyaml["agents.yaml"]
    config --> tasksyaml["tasks.yaml"]
    src --> tools["tools/custom_tool.py — example tool (unused)"]
    root --> knowledge["knowledge/user_preference.txt — example knowledge source (unused)"]
    root --> output["output/blog_post.md — final result"]
```

## Example output

Running `crewai run` executes the researcher → writer → editor pipeline:

![crewai run terminal output](docs/screenshots/terminal-run.png)

...and writes the final result to `output/blog_post.md`:

![Rendered blog post output](docs/screenshots/blog-post-output.png)

## Installation

Requires Python `>=3.10 <3.14`. Dependencies are managed with [uv](https://docs.astral.sh/uv/).

```bash
pip install uv
crewai install
```

### Configuration

Add to `.env`:

```
MODEL=gemini/gemini-flash-lite-latest
GEMINI_API_KEY=your-key-here
```

- Edit `src/blog_writing_crew/config/agents.yaml` to change agent roles/goals
- Edit `src/blog_writing_crew/config/tasks.yaml` to change what each task asks for
- Edit `src/blog_writing_crew/main.py` to change the `topic` input

## Running the project

```bash
crewai run
```

This runs the researcher → writer → editor pipeline and writes the final post to `output/blog_post.md`.

## Support

- [crewAI documentation](https://docs.crewai.com)
- [crewAI GitHub](https://github.com/joaomdmoura/crewai)
