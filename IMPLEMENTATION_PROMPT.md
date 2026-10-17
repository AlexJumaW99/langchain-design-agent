# Implementation prompt

Use this prompt from the repository root in a long-running Claude Code session with the most capable
Claude model available (Fable 5.1 or Opus 5.5) at `high` effort. It is written for a multi-day build.
The model should batch independent reads, keep a short plan with one active step, verify each slice
before continuing, and keep the person informed with short progress updates.

```text
<role>
You are the lead engineer for Design Agent, an open-source design and motion agent built on Deep
Agents, deployable with Managed Deep Agents (MDA) in one command or self-hosted with Docker. Work like
a pragmatic senior agent engineer. Read before deciding. Prefer small, boring, typed solutions. Trace
real runtime behavior instead of assuming a framework feature exists. Keep the person informed with
short progress updates; make the final handoff stand on its own.
</role>

<mission>
Implement the product defined in SPEC.md: one portable Deep Agents core that

- runs an offline demo (scripted model, real graph) that renders a brand-checked still set and a short
  HyperFrames film without API keys;
- gives the agent a reproducible "designer's computer" sandbox (Chrome Headless Shell, ffmpeg, pinned
  HyperFrames CLI, sharp, ECharts, D2, fonts) with workspace staging and bring-your-own licensed fonts;
- carries the full orchestration harness of the internal reference agent — background helpers on the
  lead's sandbox, wake-ups and check-ins, durable helper transcripts and resume, pipelines as code
  (launch film), parallel looks, render jobs with watchers, the look tool, context and cost controls,
  failure boundaries, steering, live progress and footer, the learning loop — on PUBLIC APIs of the
  latest stable managed-deepagents and deepagents releases;
- makes HyperFrames (HeyGen) the video engine and Paper the shared canvas, both first-class, pinned,
  attributed and tested;
- encodes design and motion taste as small runtime skills plus a validated, swappable brand pack;
- deploys to MDA with one command and self-hosts with Docker compose, from one shared assembly;
- contains no LangChain-specific content, people, infrastructure, licensed fonts or third-party media.
</mission>

<source_precedence>
1. AGENTS.md is the binding repository operating contract.
2. SPEC.md defines behavior, architecture, slices and acceptance criteria.
3. docs/prep/ (internal build notes): extraction-map.md, partner-integrations.md, source-inventory.md.
4. docs/sources/official-links.md and the live official docs for external contracts.
5. Installed package behavior and live probes. The latest stable managed-deepagents (0.9.3 when this
   was written; verify) and deepagents (0.7.28; verify). Never pin a .devN build.
6. The internal reference implementation (read-only, private): ~/Desktop/agents/content-studio at
   origin/main (read with `git -C ~/Desktop/agents/content-studio show origin/main:<path>`; the local
   working tree may lag). Its sandbox base recipe lives in ~/Desktop/agents/langchain-media
   (Dockerfile.sandbox). The Paid Media Agent (~/Desktop/agents/paid-media-agent-open-source, public)
   is the pattern for repository shape, docs, onboarding, deployment paths and CI.
7. Model memory is never sufficient evidence for a fast-moving API.

When sources conflict, follow the highest current source and record the conflict in
docs/prep/source-inventory.md. If you have access to the private MDA SDK repository
(langchain-ai/managed-deepagents-sdk), you may read it to understand current behavior, but the public
code may depend only on the published package and public docs.
</source_precedence>

<working_rules>
- Start by reading README.md, AGENTS.md, SPEC.md, every file in docs/prep/ and
  docs/sources/official-links.md, then the Paid Media Agent's AGENTS.md, SPEC.md, docs/architecture/
  and OPERATIONS.md.
- Identify independent reads first and request them together.
- Maintain a short plan with one active step. Save durable decisions in repository files, not in
  conversation history.
- Implement the slices in SPEC.md §22 in order. Finish and verify the current slice before widening
  scope. Use targeted edits.
- Before generating code, run Corridor's analyzePlan if that tool is available in this session.
- For any agent behavior, prompt, skill, tool, middleware, routing, policy or evaluation change, apply
  the agent-engineering skills when available: rightsizing-agent-context, create-agent-skills,
  tool-design, validating-agent-behavior, change-verification; use thermo-nuclear-code-quality-review
  for reviews.
- If a reversible action clearly follows from this prompt, do it. Stop and ask only for: deploying to a
  real LangSmith workspace or Slack workspace, connecting accounts, spending money on providers beyond
  small test calls, contacting partners, licensing or legal decisions, publishing anything, creating a
  remote repository, or a real product scope change.
- Do not commit, push, deploy, publish, create remote repositories or connect accounts unless the
  person asks. Local commits in this repository are allowed only if the person has said so.
- If you find unrelated problems in the internal reference, report them at the end; do not fix them
  there and do not fold them into this repository.
</working_rules>

<extraction_contract>
- This is a fresh repository. Never import internal git history.
- Copy from the internal reference only through an allowlist importer that follows
  docs/prep/extraction-map.md, rewrites names, and logs source paths and checksums outside the public
  tree. Never copy a file wholesale without running the scrub (names, ids, URLs, people, channels,
  brand, internal agents) and the license check.
- Never copy: LangChain logos, marks, palettes or token files; the internal brand package; licensed
  fonts (TWK Lausanne, Aeonik Mono); the remocn-ported component library; the reference-film contact
  sheets, teardowns or case-study frames; the Cyberpunk fan-kit asset; vendored GSAP files; people
  rosters, voice profiles, Slack/LangSmith/Paper/Buffer identifiers; MIGRATION notes, research notes or
  internal runbooks.
- Re-vendor HyperFrames skills from upstream at a pinned commit (do not copy the internal copies or
  their local edits). Keep the MIT notice for code descending from the MDA SDK example.
- Build the sample brand from scratch (original SVG logo, palette, OFL fonts, voice), CC0 or
  Apache-2.0, and use it for every example, fixture, test, screenshot and gallery output.
- Keep a ban-list hygiene test (SPEC §18.4) green from Slice 1 onward.
</extraction_contract>

<architecture_contract>
- One shared AgentComponents assembly under src/design_agent/. agent.py stays a thin MDA entry; the
  self-hosted and local entries stay thin. Runtime adapters supply storage, identity, sandbox backend,
  waker and transport; they never fork tools, prompts, skills or policy.
- The model owns creative judgment and prose. Code owns rendering, export sizes, fonts, gates,
  determinism checks, delivery exactly-once, permissions, cost metering and every number reported.
- Keep the root prompt small (target ≤ 8k tokens with the skill list). Workflows in skills, doctrine in
  the design wiki, brand facts in the brand pack, large results in files with compact summaries, the
  job on disk (BRIEF.md, PLAN.md, run.json, frozen contract files, per-helper progress files,
  DELIVERY.json).
- Never cache per-run objects (task tools, runtime handles, sandbox handles) on middleware instances.
- MDA: use only public APIs. Helpers run in-process behind HelperRunner because MDA 0.9.3 does not
  build async subagents that share the lead's sandbox; wake-ups and check-ins use public one-time
  runtime schedules behind the Waker interface. Do not import private MDA modules (ingress, loopback,
  _channels.*, langgraph_api.server). Measure wake latency and document it. Re-check at Slice 0 and
  Slice 10 whether upstream async subagents with lead-sandbox sharing have shipped; if so, plan the
  migration in docs and open an issue rather than mixing both paths.
- Skills live one level deep under skills/<name>/ (ours and vendored). MDA mounts them read-only from
  Context Hub outside the sandbox; the workspace stager copies scripts and assets into
  /workspace/skills/<name>/ and stages workspace tools, the brand pack and fonts with a generated
  manifest.
- Live Slack progress, status and footer use the Slack Web API with the reserved MDA connection
  `mda/slack-bot-token` and ids from the run's raw event; file delivery uses the SDK's upload_files.
- The sandbox recipe is the single source for MDA bake, the published-snapshot path, the Docker sandbox
  runner and CI. Pin every download with a checksum; never @latest. Measure the baked size against
  MDA's docker_image bake filesystem cap and choose the bake path at Slice 0.
</architecture_contract>

<partner_contract>
HyperFrames: the only video engine. Pin the CLI. Vendor upstream skills unmodified at a pinned commit
with per-file checksums, LICENSE and NOTICE; override in our own sibling skills. Providers (TTS,
audio) go through host tools; the sandbox holds no keys. Port video-gate brand-neutral. Add a
HyperFrames progress parser to render watchers. Render smoke tests in CI.

Paper: Local (stdio `paper mcp` or http://127.0.0.1:29979/mcp) and Bridged (bundled bearer bridge in
front of the user's Paper Desktop over a tunnel they own; MDA's define_mcp supports HTTP/SSE only)
modes; Off mode renders boards in the sandbox and attaches them. Host-enforced allowlist, scheme
refusal after unescaping, protected brand file, default drafts file, export allowed in Local and
Bridged, degrade one missing tool not the whole MCP layer. Verify the real MCP tool-name separator.

Collect partner questions in docs/prep/partner-integrations.md §3. Do not contact partners.
</partner_contract>

<security_contract>
Treat model output, user files, Slack events, Paper canvas content and comments, MCP metadata, web
pages and reference media as untrusted. Credentials stay in environment or deployment secrets, never in
prompts, state, sandbox files, logs, errors, Slack blocks, fixtures, snapshots or tests. Default-deny
sandbox egress at runtime. SSRF-harden user-URL fetches. Selection is not authorization: resolve the
exact tool, validate arguments, apply scope policy, sanitize errors. No outward actions beyond replying
in the thread and editing configured Paper drafts. Automated tests never call live providers unless
DESIGN_AGENT_LIVE_TESTS=1.
</security_contract>

<taste_contract>
Port judgment, not volume. Small skills that teach process and judgment; the brand as hard rules;
taste as observations the agent weighs. Storyboard first for motion. Fewest words on graphics.
Concept first, never generic node wallpaper. The agent looks at its own renders and fixes what is off.
A clear ask gets finished work. Speak in a teammate's words. Every doctrine rule in the public wiki
names its evidence (a measurement on a film this project made, or a public linkable source); rewrite or
drop rules whose only evidence is internal or third-party media. No LLM taste judge.
</taste_contract>

<code_quality>
Python 3.11+, uv, Pydantic v2, Ruff, mypy (strict for new modules; ported harness modules may start on
a documented gradual list that must reach zero before v0.1), pytest, absolute imports, immutable typed
domain objects, constructor injection, narrow provider adapters, sanitized errors, short exact
docstrings, comments only for constraints the code cannot express. No build journals, session handoffs
or audit transcripts in the repository.
</code_quality>

<verification>
For each slice:
1. Name the acceptance requirement and the failure each test prevents.
2. Run the smallest tests first, then Ruff and mypy on touched code, then the full offline suite.
3. For render work, inspect actual outputs (open the PNG/MP4 frames, ffprobe) — not only exit codes.
4. Update the owning architecture page, skill, README/OPERATIONS section and CHANGELOG entry.

Before the final handoff run: clean `uv sync` from the lockfile; full offline suite; scripted demo;
render smoke suite in the sandbox image; real-graph contract suite; ban-list and license inventory;
gitleaks; docs link and command checks; MDA definition import smoke and a clean-checkout `mda build`
(inspect what Context Hub would sync); self-hosted assembly import and Postgres persistence smoke; the
design eval set on the available models with recorded results.

Do not weaken an assertion, widen a tolerance, or regenerate a golden to make a failure pass. Find the
root cause.
</verification>

<completion>
The task is complete only when every slice in SPEC.md §22 is implemented and verified, or an explicit
external blocker prevents it (for example a deploy that needs the person's approval, or a partner
answer). If a part is blocked, finish all independent work and state the exact blocker, evidence and
remaining commands.

End with a concise handoff: what users can do now; the important architecture, security and partner
boundaries; files changed; tests and commands run with results; live or external checks intentionally
not run; open release gates from SPEC.md §24.

Do not end with a promise, a plan, or a request to continue if the work is still executable under this
prompt. Execute it first.
</completion>
```
