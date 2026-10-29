# adv-genai
adv-genaiv create# Agent lab — a working agent built on the course stack

This folder is the "develop" part of the course: the small runtime that turns the
Session 9a first call (`hello.py`) into a real agent that can call tools, observe
results, and keep going until it answers.

It is deliberately plain Python — no LangChain, no LlamaIndex — so every step is
readable. It reuses the **course `.env`**: `BASE_URL`, `API_KEY`, `MODEL`, which
means switching Groq ⇄ Ollama in `.env` switches the agent too, with no code change.

> Unlike `module01/` … `module14/`, this folder is *your* work, not course
> material. Keep it out of the course repo's commits if you plan to `git pull`
> often — or copy it into `adv-genai-work` when you create that in Session 14.

## Quickstart

```powershell
# from the repository root, with .env already configured
uv run python agent_lab/run.py "What is 1234 * 5678? Use the calculator."

# same thing, step by step
uv run python agent_lab/run.py -v "Read hello.py and summarise what it does"

# interactive
uv run python agent_lab/run.py --interactive
```

Or drive it from Python:

```python
import sys; sys.path.insert(0, "agent_lab")
from agentic_ai import Agent, AgentConfig, OpenAIChatClient, default_tools

config = AgentConfig.from_env()
agent = Agent(
    OpenAIChatClient(api_key=config.api_key, base_url=config.base_url, model=config.model),
    default_tools("."),
    max_steps=6,
)
print(agent.run("List the files here and tell me what this repo is.").answer)
```

## What's inside

```
agent_lab/
├── agentic_ai/
│   ├── agent.py      think -> tool call -> observation -> answer (the loop)
│   ├── tools.py      calculator + sandboxed read_file / write_file / list_files
│   ├── llm.py        one client for any OpenAI-compatible endpoint
│   ├── config.py     reads .env (BASE_URL / API_KEY / MODEL and aliases)
│   └── cli.py        command line: one-shot tasks, --verbose, --interactive
├── tests/            21 offline tests (scripted fake model, no key needed)
├── examples/offline_demo.py
├── conftest.py
└── run.py
```

## Tests (offline)

```powershell
uv run pytest agent_lab/tests -q
```

They use a fake chat client, so no key, no network, no model download.

## How the loop works

1. Send the task + the tool schemas to the model.
2. If it asks for a tool, run the Python function and append the result as a
   `tool` message.
3. Repeat until it answers without a tool call, or `max_steps` is reached.

Tool failures are returned as `ERROR: …` to the model, so it can recover instead
of crashing the run. File tools refuse any path outside the workspace.


