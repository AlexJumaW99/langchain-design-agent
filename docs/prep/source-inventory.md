# Source inventory

> Internal build note. Lists the private and public sources this build extracts from and the
> versions it targets. Delete `docs/prep/` before the first public push; move surviving version
> facts into `CHANGELOG.md` and `docs/architecture/`.

## Sources to extract from

| Source | Revision | Role | Access |
| --- | --- | --- | --- |
| Internal design agent ("Content Studio"), MDA app | `origin/main` @ `f87d644` (2026-10-16) | primary source for harness, middleware, tools, skills, sandbox recipe, tests | private repo `content-studio` on the builder's machine (`~/Desktop/agents/content-studio`). Read committed code with `git show origin/main:<path>`; the local working tree can lag `origin/main` |
| Internal source assets (pre-MDA app) | `origin/main` @ `ccc0eec`; asset snapshot `a94f68e` | `Dockerfile.sandbox` base-image recipe, original renderers and skills | private repo `langchain-media` (`~/Desktop/agents/langchain-media`) |
| MDA SDK example lineage | `managed-deepagents-sdk` `examples/content-studio` @ `18a218a` (MIT) | origin of the internal MDA app; its MIT notice must be preserved for any code that descends from it | private SDK repo (LangChain staff); the published package is public |
| Paid Media Agent (open source) | `4b47399` | the pattern to follow: repository shape, docs, onboarding, two deploy paths, CI, release discipline | public `github.com/langchain-ai/paid-media-agent`; local `~/Desktop/agents/paid-media-agent-open-source` |
| HyperFrames | release `v0.8.169` (2026-10-17); Apache-2.0 | video engine CLI and agent skills | public `github.com/heygen-com/hyperframes`, npm `hyperframes` |
| Paper Desktop MCP | current Desktop release | shared canvas | `paper mcp` stdio CLI or `http://127.0.0.1:29979/mcp` |

The internal agent was the subject of the "How we built our Design Agent" launch post. The public
name is **Design Agent**; "Content Studio" is the internal codename and may appear only in
`NOTICE.md` provenance.

## Baseline health of the source (measured 2026-10-17)

- `origin/main` exported with `git archive`, `uv sync --frozen --all-extras --dev`, then
  `uv run --frozen pytest -q`: **2,441 passed** in 106 s, offline, no `.env`.
- 2,190 tracked files: skills 977, workspace 868, tests 140 (122 Python files, 44,822 lines,
  1,589 test functions before parametrization), tools 57, middleware 39, docs 26, scripts 24,
  harness 18 (6,438 lines, no MDA imports), fonts 10.
- No `Makefile`, `langgraph.json`, Dockerfile, CI workflow or top-level `LICENSE` on `main`. Gates
  are listed in its `AGENTS.md` (`ruff check`, `ruff format --check`, `basedpyright`, `pytest`,
  `scripts/check_deploy_tree.py`).
- The sandbox base snapshot is built outside the repo (from `langchain-media/Dockerfile.sandbox`)
  and carries licensed fonts, the internal brand package, the video-kit and June copies of the
  HyperFrames skills. The open-source build must replace it with a public, reproducible recipe.

## Target framework versions (verify on Slice 0)

| Package | Latest stable on 2026-10-17 | Internal pin | Paid Media pin | Target |
| --- | --- | --- | --- | --- |
| `managed-deepagents` | **0.9.3** (dev: 0.10.0.dev9) | `==0.9.0.dev2` (dev build) | `>=0.6.1,<0.7` | latest stable at build time, `>=0.9.3,<0.10`; never a `.devN` pin in a release |
| `deepagents` | **0.7.28** | `>=0.7.5,<0.8` | `>=0.7.12,<0.8` | `>=0.7.28,<0.8` |
| `langchain`, `langgraph`, `langchain-anthropic`, `langchain-openai`, `langchain-mcp-adapters`, `mcp` | resolve at build | | | current compatible, explicit upper bounds on pre-1.0 packages |
| `hyperframes` (npm) | `0.8.169` | `@latest` at bake | n/a | exact pin in `sandbox/setup.sh`, upgrade through the weekly compatibility job |
| HyperFrames skills | upstream `main` at a pinned commit | June 2026 pull, hash-only lock, 93 local edits | n/a | pinned commit + per-file sha256 in `skills-lock.json`, unmodified |

The internal agent depends on private MDA modules in two places (`middleware/boundary.py` for
background wake-ups through MDA's private ingress and loopback, and `tools/connect.py` for Agent
Auth). The open-source build must use only public APIs from the latest stable release; see SPEC §7.

## Identifiers that must never reach the public repository

Scrub by category, enforced by a CI ban-list scan (SPEC §18.4):

- LangChain brand: logos, icons, marks, palettes, the token files, construction specs, graphic
  language images, brand-book Paper file ids, the internal brand package.
- People: employee names, Slack user ids, emails, voice profiles, the people roster, approver
  principals.
- Workspace and infrastructure: Slack channel and team ids, LangSmith workspace, project and
  deployment ids and URLs, snapshot names, Paper team id, Okta and sign-in hand-off details,
  managed-tool server ids, Buffer channel ids, S3/R2 hosts.
- Internal agents and programs: peer agents and their bot ids, the quote-post copilot, the launch
  ledger and its guards, internal campaigns, product glossaries, analytics workspaces.
- Licensed or third-party media: TWK Lausanne and Aeonik Mono fonts, the remocn-ported component
  library, the 70 reference-film contact sheets and teardowns, frames of LangChain films, the
  Cyberpunk fan-kit asset and terms, vendored GSAP files, third-party brand imagery in examples,
  externally sourced image galleries without a clear license.
- History: `MIGRATION.md`, `migration-notes/`, internal runbooks, trace skills, research notes.
