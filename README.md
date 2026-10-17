# Design Agent (spec-first)

On-brand design and motion from a chat thread. Built on [Deep Agents](https://github.com/langchain-ai/deepagents).
Deploy with [Managed Deep Agents](https://docs.langchain.com/langsmith/python/managed-deep-agents-overview)
or self-host with Docker. Renders with [HyperFrames](https://hyperframes.heygen.com). Designs with
[Paper](https://paper.design).

This repository is in its spec-first phase, like the Paid Media Agent before its first release. It
holds the specification and the build prompt; the implementation lands slice by slice.

| Read | For |
| --- | --- |
| [SPEC.md](SPEC.md) | product behavior, architecture, slices, acceptance criteria, open questions |
| [AGENTS.md](AGENTS.md) | the operating contract for coding agents working here |
| [IMPLEMENTATION_PROMPT.md](IMPLEMENTATION_PROMPT.md) | the prompt for the long-running build session |
| [docs/prep/](docs/prep/) | internal build notes: extraction map, partner integrations, source inventory (deleted before the public release) |
| [docs/sources/official-links.md](docs/sources/official-links.md) | primary documentation for every external contract |

## Start the build

Open this folder in Claude Code and run the prompt in `IMPLEMENTATION_PROMPT.md`, or ask:

> Read AGENTS.md, SPEC.md, docs/prep/ and IMPLEMENTATION_PROMPT.md, then implement Design Agent
> slice by slice starting with Slice 0. Stop for deployments, account connections, partner contact
> and licensing decisions.

---

Target license: Apache 2.0 · Target repository: `github.com/langchain-ai/design-agent`
