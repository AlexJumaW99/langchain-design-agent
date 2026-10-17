# Partner integrations: HyperFrames and Paper

> Internal build note. It references the internal source agent for extraction purposes. Delete
> `docs/prep/` (or move its surviving facts into public docs) before the first public push.

Design Agent is co-launched with two partners. HyperFrames (HeyGen) is the video engine and Paper is
the shared design canvas. Both integrations must be first-class, attributed, pinned, testable without
private infrastructure, and degrade gracefully when the user has not connected them.

## 1. HyperFrames (HeyGen)

### 1.1 What it is to this agent

HyperFrames turns HTML into video: a composition is a web page with timing in `data-*` attributes and
motion on a paused GSAP timeline, rendered frame by frame in Chrome Headless Shell and encoded with
ffmpeg, so the same file always renders the same video. The CLI is the agent's timeline panel:

| Command | Agent use |
| --- | --- |
| `hyperframes init --non-interactive [--example …]` | scaffold a project in `/workspace/projects/<slug>/` |
| `hyperframes lint [--json]` | catch broken timing and contract violations before a render |
| `hyperframes validate` | composition contract check |
| `hyperframes inspect [--json --samples N --at t]` | overflowing text, layout drift |
| `hyperframes snapshot --at <t>` | one frame to look at (the preview server is unreachable from Slack) |
| `hyperframes render --quality draft\|standard\|high --fps 24\|30\|60 --output <file> [--variables-file …]` | final and draft renders |
| `hyperframes add <block>` | registry blocks |
| `hyperframes capture <url>` | website-to-video source capture |
| `hyperframes transcribe --model …`, `tts`, `remove-background` | media helpers |
| `hyperframes browser ensure`, `doctor` | bake-time Chrome Headless Shell install and warm-up |

Verify every flag against the pinned CLI's `--help` during Slice 0. The internal agent used a
`--resolution square-4k` flag that is not in the CLI reference; do not carry undocumented flags.

### 1.2 v1 decisions

- **HyperFrames is the only video engine in v1.** The internal agent also has a bespoke 2D canvas
  engine ("motion-studio") that is tied to its own brand palette, fonts and logos and never calls
  HyperFrames. Do not port the engine. Port its *judgment* (the motion wiki's directing, craft,
  techniques, sound and review doctrine, scrubbed and re-evidenced) as runtime skills that drive
  HyperFrames compositions.
- **Pin, do not float.** The internal sandbox installed `hyperframes@latest` at bake time and the
  skills were a June pull with no commit pin. The open-source recipe pins an exact CLI version and an
  exact upstream skills commit; a scheduled compatibility job proposes upgrades.
- **Vendor the skills unmodified.** Pull the current upstream skill set from
  `heygen-com/hyperframes` at a pinned commit into top-level `skills/<upstream-name>/` folders (MDA
  discovers skills only one level deep), with the upstream LICENSE and a NOTICE, and record each in
  `skills-lock.json` with the commit and per-file sha256. Do not reapply the internal agent's 93 local edits: they were routing and
  brand text. Put local routing, provider overrides and brand-neutral guidance in our own skills
  that sit beside the vendored ones (for example `motion-direction`, `storyboard`, `launch-film`).
  If an override needs a vendored file changed, upstream it.
- **Use the current upstream layout.** Upstream split skills since the internal June pull
  (`hyperframes-audio`, `hyperframes-keyframes`, `media-use`). Vendor what upstream ships at the
  pinned commit; do not resurrect retired internal skills.
- **Providers stay host-side.** Upstream skills can call HeyGen TTS or audio APIs with
  `$HEYGEN_API_KEY` from inside the sandbox. The sandbox never holds keys. Route voice and music
  through host tools (`generate_voiceover`, `generate_music`, `generate_sound_effect`) and, if HeyGen
  offers an open-source key or a host-side API, add a host-tool adapter for it.
- **Render watching.** Long renders run detached (`start_render_job`) with a watcher that wakes the
  lead on exit, failure or stall. Add a HyperFrames progress parser (the internal watcher had parsers
  for remotion, ffmpeg, json and regex, but none for HyperFrames, and defaulted to `remotion`).
  Ask HeyGen for machine-readable render progress (see questions).
- **Determinism gate.** Keep `video-gate` (rejects `Math.random(`, `Date.now(`, `new Date()`,
  timers, `requestAnimationFrame`, unpaused timelines and other seek-breaking patterns) and make it
  brand-neutral: it must not call a brand linter with hard-coded tokens.
- **Attribution.** README "Built with" section, NOTICE entry, the HyperFrames skills LICENSE, and a
  pull request to add Design Agent to HyperFrames' `ADOPTERS.md` once the repository is public and
  HeyGen agrees.

### 1.3 Sandbox recipe (HyperFrames parts)

Node 22 LTS (verify HyperFrames' supported range), ffmpeg, Chrome Headless Shell via
`hyperframes browser ensure` into a fixed cache dir, the pinned `hyperframes` CLI on PATH, fonts
registered with fontconfig, `text-rendering: geometricPrecision` defaults in templates. The recipe
fails the bake if `hyperframes lint` and a 1-second draft `render` of the bundled fixture composition
do not succeed and produce a non-empty MP4 with the expected duration and dimensions (ffprobe).
Do not install two Chromiums unless a renderer needs both (the internal image carried Playwright's
Chromium and HyperFrames' headless shell); prefer one and point other renderers at it.

### 1.4 HyperFrames acceptance tests

- Vendored skills match `skills-lock.json` (upstream repo, commit, path, sha256 per file).
- Fixture composition lints, validates, snapshots at t=1.0s, and renders a 2 s 1920x1080 MP4 in the
  sandbox; ffprobe confirms duration within one frame and a non-blank first and last frame.
- `video-gate` rejects each banned pattern with a fixture and passes the clean fixture.
- Watcher parses HyperFrames progress (or a heartbeat when none is emitted) and wakes the lead on
  exit 0, non-zero exit, and a stall.
- A source-blind eval: asked for a 15 s teaser, the agent lints and snapshots before rendering and
  delivers one final MP4.

## 2. Paper

### 2.1 What it is to this agent

Paper is a design canvas people and the agent both edit. The agent writes HTML onto the canvas, where
it becomes layers a designer can grab and move, and reads boards, tokens, comments and screenshots
back. It is the default surface for new visual work that people will want to point at and nudge
(layouts, storyboards, thumbnails, campaign graphics), while renders still happen in the sandbox.

### 2.2 Transport facts

- Paper's MCP server runs inside Paper Desktop. Desktop installs a `paper` CLI; the recommended
  transport is the stdio server `paper mcp`. The legacy HTTP endpoint `http://127.0.0.1:29979/mcp`
  is served only while a file is open.
- There is no documented hosted or token-based remote MCP. Free plans have MCP call limits that Pro
  lifts.
- The internal agent ran Linux Paper Desktop under Xvfb on an always-on cloud sandbox, signed in
  once as an operator through a custom sign-in hand-off app, fronted by a team-scoped bearer bridge.
  That works but is heavy, depends on undocumented Linux Desktop support, and is not a reasonable
  default for open-source users.

### 2.3 Connection modes (v1)

| Mode | When | How |
| --- | --- | --- |
| **Local** (default for the CLI and Studio) | the agent runs on the same machine as Paper Desktop | stdio `paper mcp` (preferred) or `http://127.0.0.1:29979/mcp`; host-side MCP client; no bridge |
| **Bridged** (self-host or MDA, advanced) | the agent runs elsewhere and the user runs Paper Desktop on a machine they control | the bundled `paper-bridge` (Starlette MCP proxy) in front of the local MCP, reachable over a tunnel the user owns, with a ≥32-byte bearer token, DNS-rebinding allowlist, team check and tool allowlist; the token is an MDA connection or self-host secret |
| **Hosted worker** (experimental, documented recipe only) | a team wants an always-on shared Paper session | the Linux-Desktop-under-Xvfb worker pattern, documented as a recipe with its caveats, not wired by default; replace with a partner-hosted endpoint when Paper ships one |
| **Off** | Paper not configured | boards render as PNG/PDF in the sandbox and are attached to the thread; the agent says Paper is not connected once and continues |

### 2.4 Tool policy (host-enforced)

Port the internal scope middleware, generalized:

- Allowlist reads (`get_basic_info`, `get_selection`, `get_node_info`, `get_children`,
  `get_screenshot`, `get_jsx`, `get_tree_summary`, `get_computed_styles`, `get_fill_image`,
  `find_nodes`, comment-thread reads, `get_tokens`, `get_font_family_info`, `get_guide`), edits
  (`create_artboard`, `create_page`, `write_html`, `set_text_content`, `rename_nodes`,
  `update_styles`, `duplicate_nodes`, `move_nodes`, `finish_working_on_nodes`), files (`list_resources`
  or its current name, `open_file`, `create_file`) and, in Local and Bridged modes, `export` and
  `export_combined_pdf` (the internal "Paper can't export" limit was a property of the cloud bridge,
  not of Paper).
- Deny deletes, token writes and page/file reorganization by default; they are opt-in per
  deployment and still never touch the protected brand file.
- Deny `paper-gen:` (AI image generation billed to the user's Paper plan) unless the operator
  enables it; deny `paper-asset:` and `file:` schemes after HTML-unescaping arguments.
- `PAPER_BRAND_FILE_ID` (optional) is read-only. `PAPER_DRAFTS_FILE_ID` (optional) is the default
  target when a call omits `fileId`.
- Re-check the allowlist against the server's advertised tools at connect time and degrade one
  missing tool, not the whole MCP layer. Do not pin `include_tools` in a way that turns a Paper-side
  rename into a total MCP outage. Paper renamed `list_files` to `list_resources` and made
  `get_font_family_info` file-scoped in October 2026; tool names are not yet a stable contract.
- One failing MCP server must never remove the others or the agent's other tools.

### 2.5 Workflow contracts

- Storyboards: frames go to a Paper page (editable twin) and the sign-off board is exported (Local or
  Bridged) or rendered in the sandbox (Off or Hosted) and attached to the thread. The thread always
  gets an image people can approve.
- Brand import: the brand-onboarding coding-agent skill can read a Paper file's tokens, palette and
  type nodes (`get_tokens`, `get_computed_styles`, `get_jsx`) to draft `workspace/brand/tokens.json`
  and `graphic-language.md`. A person reviews the result.
- Receipts: record Paper file id, page id and link in the project's `run.json`; never store tokens.
- Always call `get_guide(topic="paper-mcp-instructions")` once per session, pass `fileId`
  explicitly, check with `get_screenshot`, and call `finish_working_on_nodes` when done.

### 2.6 Paper acceptance tests

- Scope middleware: allowlist, denied tools, scheme refusal after unescaping, brand-file write and
  clone refusal, default file fill, missing-tool degradation (unit tests against a fake MCP server).
- Bridge: bearer compare, rebinding allowlist, path guard, team check (unit tests).
- Contract test with a fake Paper MCP through the real graph: storyboard written to a page, export
  attached, receipt recorded.
- Opt-in live test (developer machine with Paper Desktop open): read `get_basic_info`, write one
  artboard to a scratch file, screenshot it, export a PNG, clean up.

## 3. Partner questions

### For HeyGen (HyperFrames)

1. Confirm the license covering the skills folder (the package and repository are Apache-2.0).
2. Versioned skill releases or a recommended commit pin; is `hyperframes skills check` the supported
   drift check?
3. Recommended way to override routing or providers (host-side TTS) without forking skill files.
4. Where bundled SFX audio and the matting model live (the internal pull lacked the binaries).
5. Machine-readable render progress for watchers (JSON lines on stderr or a progress file).
6. Is `@latest` plus `--docker` the recommended reproducibility path, or a pinned CLI?
7. Registry pinning: should the agent pin the registry to a tag instead of `main`?
8. An open-source key or free tier for HeyGen TTS/audio, and the status of cloud render targets.
9. Telemetry defaults inside sandboxes and how to disable them.
10. GSAP licensing guidance: the upstream graphic-overlays skill vendors GSAP (GSAP Standard License);
    should open-source users load it from a CDN at render time instead?
11. Co-marketing: ADOPTERS listing, a joint example in their catalog, and a quote for the launch post.

### For Paper

1. A hosted or remote MCP, or a headless/service-account mode, for agents that run in the cloud.
2. Multi-user authorization and team scoping for a shared agent.
3. How `export` and `export_combined_pdf` bytes should reach a remote agent.
4. A supported raster import path without `paper-asset:` or `file:` URLs.
5. Tool-name stability and a version or capability endpoint.
6. Pagination for files and directories beyond 200 entries.
7. Targeting a file without changing the user's active Desktop file.
8. Agent call limits per plan and `paper-gen:` pricing.
9. Official Linux Desktop support (needed only for the hosted-worker recipe).
10. Co-marketing: a Paper integration page, a template file for the sample brand, and a quote.
