# Extraction map: internal agent → Design Agent

> Internal build note. Paths on the left refer to the private source repository
> (`content-studio` @ `f87d644`, read with `git show origin/main:<path>`). Never copy a file
> wholesale without passing it through the scrub and license checks in SPEC §17. Delete
> `docs/prep/` before the first public push.

Actions:

- **KEEP**: port with light edits (names, config, imports). Behavior unchanged.
- **GENERALIZE**: port the mechanism; remove company content; move constants to config or the brand
  pack; rewrite examples against the sample brand.
- **REWRITE**: the idea survives, the code or text does not (written fresh for the public repo).
- **REPLACE**: superseded by a public upstream feature (latest MDA, Deep Agents, HyperFrames, Paper).
- **EXCLUDE**: not part of the open-source product. Never copied.
- **DEFER**: valuable, out of v1 scope; listed as a roadmap item with its extension point.

Target paths are relative to the new repository root.

## 1. Agent definition and assembly

| Source | Action | Target | Notes |
| --- | --- | --- | --- |
| `agent.py` (245) | REWRITE | `agent.py` (thin, ≤40 lines) + `src/design_agent/assembly.py` | One `build_agent_components(settings, runtime)` shared by MDA, self-host and CLI, as in the Paid Media Agent. Social, quote, LinkedIn, Notion, Webflow, handoff wiring and card-action maps are removed. Feature flags become typed `Settings` fields. |
| `subagents.py` (188) | GENERALIZE | `src/design_agent/assembly.py` (`build_subagents`) | Keep one worker declared at three effort levels plus `general-purpose`. Rewrite `WORKER_PROMPT` without the internal name, cost figures or knowledge paths; keep its contracts (read named files first, own only assigned files, progress file, `report_progress`, image budget, stop at time or quality bar, reviewers do not fix, never deliver). Effort levels must map to whatever the configured model supports (capability registry). |
| `models.py` (544) | GENERALIZE | `src/design_agent/models/` | Direct providers by default (`provider:model`), LangSmith gateway as an optional preset (`langsmith:provider/model`), as in the Paid Media Agent. Remove the hard-coded gateway origin. Keep: exact-id capability registry (vision, tool search, effort levels, output ceilings, thinking/cache behavior), the Opus fallback profile logic, stall timeout, SDK retries at 0 with our retry middleware, cache TTL per role. Prune the internal model list to verified public ids. |
| `instructions.md` (76) | REWRITE | `instructions.md` | Generic identity ("a design agent for your team"), delegation contract, delivery contract, approval rules, translation rule (speak in a teammate's words, never tool names), storyboard-first rule, brand source precedence. No peer agents, no Slack ids, no people roster, no internal deployment notes. Keep under ~80 lines; everything else lives in skills. |
| `identity.py` | KEEP | `identity.py` | `define_identity(auth=auth.langsmith_api_key())`; document MDA's Supabase identity for private per-user conversations. |
| `memory.py` (48) | GENERALIZE | `memory.py.example` (off by default) | Per-user memory off by default (privacy); shared agent memory opt-in. Document the trade-offs as the Paid Media Agent does. |
| `NOTICE.md`, `licenses/managed-deepagents-sdk-MIT.txt` | REWRITE / KEEP | `NOTICE.md`, `licenses/` | Provenance for code descending from the MIT SDK example; HyperFrames skills (Apache-2.0); OFL fonts; ECharts (Apache-2.0); vtake-skills (MIT) if graphic-overlays ships; D2 (MPL-2.0, binary in image only). |

## 2. Harness (orchestration core)

The internal `harness/` package (18 files, 6,438 lines) imports only the standard library,
langchain, langchain_core and langgraph. It is the main value to carry.

| Source | Action | Target | Notes |
| --- | --- | --- | --- |
| `harness/events.py`, `policy.py`, `inbox.py`, `notices.py` | KEEP | `src/design_agent/harness/` | Event bus, attention policy (wake now / next turn / log only), coalesced wake-up inbox with retries, notice text. |
| `harness/tasks.py`, `ledger.py`, `wakeups.py`, `protocols.py` | KEEP | same | Background task records and heartbeats, self-scheduled check-ins, `HelperRunner`/`SandboxExec` protocols. Add a `Waker` protocol (SPEC §7.4). |
| `harness/durable.py` | KEEP | same | Helper transcript checkpoints and `resume_task`. Re-verify against the latest Deep Agents: if it now checkpoints background subagents natively, REPLACE and delete ours (SPEC §7.3). |
| `harness/workflow.py` | KEEP | same | Pipelines as code (memoized steps, `ask` operator questions, resume). |
| `harness/branches.py` | KEEP | same | Parallel creative "looks" (folders per look, pick and merge with backup). |
| `harness/offload.py`, `watch.py` | KEEP | same | Detached render jobs and log watchers. Add a HyperFrames progress parser and make the default parser `heartbeat`, not `remotion`. |
| `harness/cache.py`, `context.py` | KEEP | same | Cache TTL, tools breakpoint, prefix watch, stagger gate; server-side tool-result clearing; image-aware token counts. Re-verify against current provider APIs and Deep Agents summarization. |
| `harness/meter.py` | GENERALIZE | same | Move the price table to `config/prices.toml` with a date and source link; cost is an estimate. |
| `harness/learning.py` | KEEP | same | Per-skill observation store (save, read, rate, merge, decay). Optional feature, on by default once the store exists. |
| `docs/harness/*.md` | GENERALIZE | `docs/architecture/orchestration.md` and siblings | Public explanations of each primitive, with the upstream mapping table. |
| `tests/harness/*`, `tests/test_{background,look,looks,learning,watch,pipeline,harness_surface,subagents}.py` (244 tests) | KEEP | `tests/unit/harness/`, `tests/contract/` | Rewrite fixtures that mention internal names. |

## 3. Middleware

| Source | Action | Target | Notes |
| --- | --- | --- | --- |
| `middleware/background.py`, `watch_tools.py`, `pipeline_tools.py`, `looks_tools.py`, `learning_tools.py`, `look.py`, `meter.py` | GENERALIZE | `src/design_agent/middleware/` | Type against a runtime adapter interface, not `ManagedDeepAgentRuntime` directly. `LOOK_MODEL` from config. Wake-ups go through `Waker` (MDA adapter on public APIs; self-host adapter in-process). |
| `middleware/boundary.py` (484) | REWRITE | `src/design_agent/assembly.py` + `runtime/mda.py` | Assembly of guards and parent-only middleware, subagent construction. Remove every private MDA import (`_channels.slack.middleware`, `_ingress`, `_ingress_runs`, `_loopback`, `langgraph_api.server`). |
| `middleware/model_retry.py`, `step_budget.py`, `image_clamp.py`, `tool_error_boundary.py`, `tool_history.py`, `drop_base_prompt.py`, `today.py` | KEEP | same | Model failure boundary (errors never end a run silently), step budget wrap-up, image MIME and size clamp, tool error boundaries, stale `tool_reference` repair, base prompt drop, date note. |
| `tools/disclosure.py` | GENERALIZE / REPLACE | `src/design_agent/middleware/tool_disclosure.py` | Provider-native deferred tool search for verified Anthropic/OpenAI models; `LLMToolSelectorMiddleware` otherwise (Paid Media pattern). Keep the search-header trick for deferred MCP tools. `ALWAYS_SHOWN_SERVERS` from config. Prefer the upstream middleware where it now covers this. |
| `middleware/progress.py`, `phrasebook.py`, `footer.py`, `agent_session.py`, `steering.py` | GENERALIZE | `src/design_agent/middleware/` (+ self-host Slack adapter) | Streaming progress per agent, plain-language step names, reply footer (agents running, time, cost, trace link host from config), Slack agent status, mid-job steering. Phrasebook loses Notion/Hex/Webflow/GTM/Buffer entries. Prefer native MDA Slack streaming/status APIs where the latest SDK provides them (SPEC §12). |
| `middleware/workspace.py` (467) | GENERALIZE | `src/design_agent/middleware/workspace.py` | Stage workspace tools, the active brand pack and fonts into the sandbox with a manifest and sha check; licensed-font setup becomes data-driven from `brand/fonts.toml` (no hard-coded families, no woff2 sha constants). managed-deepagents 0.9.3 has no `deployment_files` or declarative sandbox file sync; keep the stager and upload through `runtime.backend.upload_files`. Skills are mounted read-only from Context Hub outside the sandbox, so the stager must also copy skill scripts into `/workspace/skills/<name>/`. |
| `middleware/paper_scope.py` | GENERALIZE | `src/design_agent/middleware/paper_scope.py` | See partner-integrations §2.4. |
| `middleware/memory_policy.py`, `memory_hot_cap.py`, `memory_learnings.py`, `owner_memory.py` | GENERALIZE | `src/design_agent/middleware/memory/` | Layout allowlist and preference-file shape become config; hot cap stays (200 lines / 12k chars). Only installed when memory is enabled. |
| `middleware/tracing.py`, `trace_labels.py` | GENERALIZE | `src/design_agent/middleware/tracing.py` | Generic run tags (`trigger`, `surface`, `delivered`); no user ids in tags by default. |
| `middleware/slack_actions.py` | DEFER | — | Code-drawn cards are only needed for approvals in the excluded publishing module. |
| `middleware/routines.py` | DEFER (v0.2) | `src/design_agent/routines/` | Housekeeping sweep stays in v1 (§5); user-owned routines are a v0.2 slice with generic sources only. |
| `middleware/asked_only.py`, `bot_callers.py`, `agent_requests.py`, `card_replies.py`, `lazy_connection.py`, `publish_ledger.py`, `notion_scope.py`, `webflow_scope.py` | EXCLUDE (v1) | — | Internal channels, peer agents, quote cards, Managed Tools, company publishing. The exactly-once publish ledger is a documented pattern for a future publishing plugin. |

## 4. Tools

| Source | Action | Target | Notes |
| --- | --- | --- | --- |
| `tools/media.py` (707) | GENERALIZE | `src/design_agent/tools/media.py` | `generate_image` (OpenAI direct by default; model id configurable and passed through, never allowlisted; transparency reported), `generate_voiceover` / `generate_sound_effect` / `generate_music` (ElevenLabs, optional), `transcribe_audio` (local `hyperframes transcribe` default; Groq optional). Remove `fetch_epidemic_asset` (Epidemic excluded in v1). Host-side only; results dropped into `/workspace/...` as files. |
| `tools/files.py`, `projects.py`, `flags.py` | KEEP | `src/design_agent/tools/` | Binary I/O, `list_recent_projects`, `get_render_status`, flag parsing (typed settings replace most flags). |
| `tools/mcp.py` | GENERALIZE | `src/design_agent/tools/mcp.py` + `config/mcp.example.toml` | Config-driven servers. Zero servers is valid (the internal import raised "No MCP server is configured"). Ships: Paper (modes per partner doc), optional Exa (web research for references), optional GitHub read-only (for `pr-to-video`). One failing server never removes the others. |
| `tools/paper.py`, `scripts/paper_mcp_bridge.py` | GENERALIZE | `src/design_agent/tools/paper.py`, `src/design_agent/paper_bridge/` | Local and Bridged modes; the cloud-worker auth flow is not wired by default. |
| `scripts/paper_cloud_worker.py`, `paper_oauth_bootstrap.py`, `paper_oauth_macos.swift`, `docs/runbooks/paper.md` | REWRITE (docs only) | `docs/partners/paper-hosted-worker.md` | Recipe and caveats, no operator identities, no sign-in hand-off app. |
| `tools/slack.py`, `slack_read.py`, `slack_ui.py` | GENERALIZE | `src/design_agent/tools/slack.py`, `surfaces/slack/` | Thread and file reads as the asker could, file fetch; remove `ASKED_ONLY` constants. Cards deferred. |
| `tools/grounding.py` (Drive/Docs) | DEFER | — | Optional host tool later; document how to add a read-only grounding tool to the assembly. |
| `tools/agents.py`, `connect.py`, `managed_tools.py` (+ catalog), `hosted_assets.py`, `person_memory.py`, `voices.py`, `compose_quote.py`, `draft_quote.py`, `quote_card.py`, `trace_flag.py`, `ledger/*`, `social/*`, `routines.py`, `routine_options.py`, `routine_tick.py`, `schedule_run.py` | EXCLUDE (v1) / DEFER | — | Peer handoffs, Managed Tools, public asset hosting, quote copilot, launch ledger, social publishing. Routines DEFER to v0.2 with generic sources (RSS/Atom, web page, Slack links). |

## 5. MDA project declarations

| Source | Action | Target | Notes |
| --- | --- | --- | --- |
| `channels/slack.py` | GENERALIZE | `channels/slack.py` | Name, description, icon, color from settings (Paid Media pattern). Keep the @mention-only trigger option as a setting, default to the platform's default behavior. |
| `channels/reactions.py` | EXCLUDE | — | Keep only the platform's default acknowledgement (eyes). |
| `sandbox/__init__.py` | REWRITE | `sandbox/__init__.py` | Public base image (`docker_image=` a pinned public base) plus `setup.sh` bake; size from settings (default 8 vCPU / 16 GiB for motion; document smaller sizes for stills-only use); idle TTL; default timeout. No snapshot names. |
| `sandbox/setup.sh` | GENERALIZE | `sandbox/setup.sh`, `sandbox/Dockerfile` | Fold in the public parts of the internal base image (Node, ffmpeg, Chrome Headless Shell via `hyperframes browser ensure`, sharp, ECharts, D2, open fonts, fontconfig) so the whole computer is reproducible from the repository. Pin every download with a sha256. Drop `yt-dlp` (third-party video download is not a product feature), Playwright's second Chromium (point renderers at one browser), licensed fonts, brand assets and video-kit. Fail the bake on a lint + draft-render smoke test. |
| `schedules/housekeeping.py` | KEEP | `schedules/housekeeping.py` | Sweep helpers, watches and wake-ups killed by a restart; ends in code before any model call. |
| `pipelines/__init__.py`, `pipelines/launch_film.py` | GENERALIZE | `src/design_agent/pipelines/` | Keep the launch-film pipeline; paths point at the public `launch-film` skill. |
| `asset-manifest.json`, `base-image-assets.json`, `scripts/check_deploy_tree.py`, `scripts/refresh_base_assets.py` | REWRITE | `scripts/build_manifest.py`, `scripts/check_deploy_tree.py` | Generated manifest (no hand-pinning), private fonts declared in the brand pack, check that required inputs exist before deploy. No base-image upload optimization in v1 (the public recipe does not bake project files). |

## 6. Runtime skills

| Source skill | Action | Target skill | Notes |
| --- | --- | --- | --- |
| 16 HyperFrames skills (`hyperframes`, `-core`, `-cli`, `-animation`, `-creative`, `-media`, `-registry`, `faceless-explainer`, `slideshow`, `website-to-video`, `pr-to-video`, `music-to-video`, `embedded-captions`, `graphic-overlays`, `general-video`, `remotion-to-hyperframes`) | REPLACE | `skills/<upstream-name>/` (top level, listed in `skills-lock.json`) | Re-vendor from upstream at a pinned commit, unmodified, with LICENSE and NOTICE. Do not carry the 93 local edits. Remove the CDPR fan-kit asset if upstream still ships it (non-commercial terms) and any vendored GSAP file (load GSAP from the image instead). |
| `motion-studio` (engine + wiki) | GENERALIZE (wiki) / EXCLUDE (engine) | `motion-direction` (+ `references/` pages) | Port the wiki doctrine (directing, craft, techniques, sound, review) as HyperFrames-oriented guidance. Drop the canvas engine, `brand.js` palettes, mark loaders and font gates. Re-evidence rules (SPEC §11.3). |
| `launch-film` | GENERALIZE | `launch-film` | Keep phases, sizing tables, decomposition rules, effort-per-role table, every brief template, `orchestration.md`, `storyboard.md`. Remove `case-studies/agent-platform/` and `case-studies/engine-teaser/` and every line citing them; replace with a case study the agent makes for the sample brand during Slice 9. |
| `film-breakdown` | KEEP | `film-breakdown` | Measure any film the user supplies (cuts, contact sheets, motion energy, beat grid, audio). |
| `video-references` | REWRITE | `reference-study` | The method, not the library: study a user-supplied film frame by frame, write a teardown into Git-ignored `workspace/references/`, and propose observations or skill edits for a person to approve. |
| `video-storyboard` | GENERALIZE | `storyboard` | Brand paths → brand pack. |
| `ad-creative` | GENERALIZE | `stills` (or keep name `ad-creative`) | `html2png.mjs` must read fonts from the brand pack and render with open fonts when no licensed fonts are configured. |
| `data-viz` | GENERALIZE | `data-viz` | Move the hard-coded token object into brand-pack reads; replace examples that name people or LangChain diagrams. |
| `image-ops`, `gif-edit`, `visual-verify`, `film-breakdown` | KEEP | same | Generic. |
| `gpt-image` | GENERALIZE | `image-generation` | Keep craft; remove gallery entries without a clear license; brand rubric reads the brand pack. |
| `score-video` | GENERALIZE | `sound` | ElevenLabs adapters optional; synthesized cue library kept; remove Epidemic and Gemini paths. |
| `brand-design` | GENERALIZE | `brand-design` | Keep the concept gate, ground/accent/type framework; brand facts come from the brand pack. |
| `media-production` | GENERALIZE | `media-production` (router) | Request routing, storyboard-to-render process, delivery contract, Paper routing rule. |
| `content-brief` | GENERALIZE | `content-brief` | Intake for vague requests; no Notion campaign docs. |
| `youtube-thumbnail` | GENERALIZE | `thumbnails` | Craft and gates kept; `channel-today.md` (internal channel and people) removed. |
| `youtube-upload`, `youtube-workflow` | DEFER | — | Upload needs OAuth and publishing policy (v0.2 publishing plugin). |
| `paper-design` | GENERALIZE | `paper-design` | Connection-mode aware; no team/brand-book ids. |
| `routines` | DEFER (v0.2) | `routines` | Ships with the routines slice. |
| `social-learning`, `social-onboarding`, `compose-post`, `technical-blog` | EXCLUDE (v1) | — | Copy and social are outside the design scope; `compose-post`'s human-voice protocol may inform `brand/voice.md` guidance. |
| `analytics`, `community-publishing`, `draft-queue`, `social-amplification`, `quote-ideas`, `langchain-media-render`, `langchain-brand-guidelines`, `video-component-library` | EXCLUDE | — | Internal, trademarked or unlicensed. |

## 7. Workspace

| Source | Action | Target | Notes |
| --- | --- | --- | --- |
| `workspace/tools/video-gate.mjs`, `artifact-contract.mjs`, `text-fidelity.mjs`, `audio-lint.mjs`, `fetch-asset.mjs` (SSRF-hardened), `html2pdf.mjs`, `review-thumb.mjs`, `film-analysis.mjs`, `film-probe.mjs`, `video-score.mjs` | KEEP / GENERALIZE | `workspace/tools/` | Rename the `--lc-*` token prefix to a neutral `--brand-*`; remove the font name in `html2pdf.mjs`. `video-gate` stops calling brand-lint implicitly. |
| `workspace/tools/brand-lint.mjs` (1,141) | GENERALIZE | `workspace/tools/brand-lint.mjs` | Add `--tokens <path>` (default the active brand pack); move font-stack, casing and replacement-color rules into the token JSON; delete LangChain product casing rules. |
| `workspace/knowledge/brand/*`, `langchain-tokens.*`, `paper-reference.json` | REWRITE | `examples/brands/example-studio/`, `workspace/brand/README.md`, `src/design_agent/brand/schema.py` | Keep the token *schema* (`design.allowedColors`, `extendedPalette`, `officialAssetColors`, `bannedColors`, `allowedFontTokens`, `knownOffBrandFonts`, `maxRectRadiusPx`, `creativeTypography`, `diagramConstruction`, `copy.*`) as a documented, validated JSON schema. |
| `workspace/knowledge/platforms/platforms-and-content-types.md`, `brand-applications.md` | GENERALIZE | `skills/design-wiki/platforms.md` | Platform sizes and specs are generic; drop the internal email footer. |
| `workspace/knowledge/references/design-tooling-figma-paper.md` | GENERALIZE | `docs/partners/paper.md` | Public facts only. |
| `workspace/knowledge/strategy/*` | GENERALIZE (frameworks) / EXCLUDE (examples, voice corpus) | `skills/design-wiki/` | Intake framework and quality rubric survive; internal examples do not. |
| `workspace/knowledge/campaigns/*`, `references/langchain-products-glossary.md`, `hex-analytics-workspace.md`, `analytics-and-self-optimization.md`, `team/people.json` | EXCLUDE | — | Internal. |
| `workspace/video-kit/r/` (115 remocn ports), `references/` (70 films), `case-studies/` | EXCLUDE | — | Unlicensed or third-party media. `video-kit/tools/ref-analyze.py` → KEEP as `skills/film-breakdown/scripts/`. |
| `workspace/video-kit/VIDEO-DIRECTION.md`, `VIDEO-DESIGN-LANGUAGE.md`, `motion/` presets | GENERALIZE | `skills/motion-direction/` | Grammar kept; brand-correctness layer reads the brand pack; motion presets renamed off `LCMotion`. |
| `workspace/skills-lock.json` | REWRITE | `skills-lock.json` | Add `commit`, `license`, per-file `sha256`; CI verifies. |
| `fonts/*` (OFL) | KEEP | `fonts/` | Keep `LICENSES.md` and `OFL-1.1.txt`; rewrite `README.md` and `MDA-BUILD.md`. |

## 8. Tests, scripts and docs

| Source | Action | Target |
| --- | --- | --- |
| Harness, guards, Slack surface, media, memory tests (≈770 tests) | KEEP / GENERALIZE | `tests/unit`, `tests/contract` |
| MCP and Paper scope tests | GENERALIZE | `tests/unit/test_paper_scope.py`, `tests/unit/test_paper_bridge.py` |
| Workspace and assets tests (snapshot name, font pins) | REWRITE | `tests/unit/test_workspace_stager.py`, `tests/render/` |
| Social, quote, ledger, voice, handoff tests (~25 files, ~420 tests) and `tests/fixtures/ledger/*` | EXCLUDE | — |
| `scripts/evals/routines_eval.py` | DEFER | v0.2 routines slice (remove its private MDA import) |
| `scripts/traces.py`, `local_probe.py`, `reactions_eval.py`, `voice/*`, `ledger/*`, `probe_managed_tools.py` | EXCLUDE | — |
| `docs/design/tool-disclosure.md`, `docs/design/memory.md`, `docs/harness/*`, `docs/local-testing.md` | GENERALIZE | `docs/architecture/*`, `docs/development.md` |
| `docs/design/social-*`, `qt-copilot.md`, `personal-posting.md`, `scheduling-migration.md`, `docs/research/*`, `docs/runbooks/social-rollout.md`, `MIGRATION.md`, `migration-notes/*`, `.claude/skills/content-studio-traces`, `.cursor/rules/*` | EXCLUDE | — |
