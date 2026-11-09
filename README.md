# omp-agent-extensions

Local extensions for omp that compose two Pi extensions:

- [`pi-context`](https://www.npmjs.com/package/pi-context) — agentic context management
- [`pi-goal-x`](https://www.npmjs.com/package/pi-goal-x) — persistent goals

All three extension files are loaded directly by omp; there is no build step.

## Design notes

[`README.html`](README.html) is a self-contained Chinese design document (v1.0, 2026-11) written for non-programmers and new contributors. It explains why `vibe-workers.ts` is built this way, including the reasoning behind its context management, host adaptation, and gates. Download it and open it in a browser to read the formatted version; GitHub shows only its source.

## Extensions

### `vibe-workers.ts`

- Reads the context-management skill body from `node_modules/pi-context/skills/` and appends it to the system prompt. omp does not discover skills bundled inside extension dependencies, so the skill would otherwise never reach the model.
- Projects the worker directive into user and tool-result messages at request time. Stored history stays unchanged.
- Adapts `pi-context` to omp's extension API: command-context acquisition, tree navigation from non-command hooks, session-manager views, and `write xd://context_compact` presented as a direct `context_compact` call.
- Adapts `pi-goal-x`: holds guided goal drafts until the new tool profile is active, routes follow-up drafts as steers when the session is idle, and keeps `/goal` on the `/goal-x` alias.

### `editor-theme-compat.ts`

Compatibility shim for `pi-goal-x` 0.34.1 against omp 18.16.0. `pi-goal-x` calls the two-argument `Editor` constructor from `pi-tui` 1.5.0, while omp's bundled `@oh-my-pi/pi-tui` 18.16.0 takes one argument. The shim gives the TUI class the `EditorTheme` surface the newer `Editor` reads, rebuilding each getter from the host's `getEditorTheme()` so theme switches take effect on the next read.

### `tool-call-gate.ts`

Adds two required arguments to every tool schema in each provider request:

- `how-many-more-workers-are-allowed-without-hitting-mythical-man-month` — integer.
- `must-think-in` — string enum with the single value `zh-cn`.

The `before_provider_request` hook injects the arguments into the tool
definitions for OpenAI-compatible, Anthropic, Google, and Bedrock request
shapes. The model therefore sees them as required. The `tool_call` hook removes
them from the execution input, so built-in tools keep their original argument
contracts.

## Install

Requires omp 18.16.0, `pi-context` 2.3.0, and `pi-goal-x` 0.34.1.

```sh
mkdir -p ~/.omp/agent/extensions
cp *.ts package.json ~/.omp/agent/extensions/
cd ~/.omp/agent/extensions
npm install
```

`node_modules/` is intentionally not committed.
