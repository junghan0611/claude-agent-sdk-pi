# AGENTS.md

## Mission

This repository exists to connect **pi** to **Claude Code** through **ACP** with the smallest possible amount of custom glue.

It is **not** a project to recreate Claude Code inside pi.
It is **not** a place to build a second harness.
It is **not** a place to accumulate protocol-shim logic because “one more compatibility fix” looks convenient.

The intended architecture is:

```text
pi
  -> this extension (thin ACP client)
    -> claude-agent-acp
      -> Claude Code
        -> native Claude config / MCP / skills / PATH
```

## Primary Goal

Keep the bridge thin, observable, and reliable.

If a change improves correctness by removing custom logic, that is usually aligned with the project.
If a change adds a new translation layer, shadow state machine, or semantic rewrite, it is usually suspect.

## Current Reality

- The repository name is still `claude-agent-sdk-pi`
- The provider ID is still `claude-agent-sdk`
- Those names remain for compatibility
- The runtime architecture is now ACP-first
- The preferred direction is increasingly **non-append**: let Claude Code load its own context where possible

Before making changes, read `README.md` first. The README history explains **why** this project pivoted and why the non-append / Claude-side capability model matters.

## Key Architectural Clarification

Do **not** assume that useful capabilities must be reintroduced through pi-native tool execution.

A valid target model is:

- pi owns harness UX and orchestration
- Claude Code owns native capability loading and execution
- this repository exposes transport, visibility, and session discipline

That means the bridge does **not** need to execute every tool itself.
But it **does** need to surface what Claude is doing so the user is not flying blind.

## What This Repository Should Own

This repository may own:

- pi provider registration
- ACP subprocess lifecycle
- ACP initialization and session management
- prompt forwarding into ACP
- ACP session-update to pi-event mapping
- visibility for Claude-side tool execution
- history/session invalidation logic when pi and ACP state diverge
- cancellation, shutdown, and diagnostics for the bridge itself

## What This Repository Should Not Own

Do **not** casually add back:

- prompt reconstruction from full pi conversation history as the default mechanism
- tool result ledgers that re-inject previous execution state
- large tool-name or tool-argument translation systems
- a parallel session model meant to “fix” Claude behavior
- emulation of Claude Code internals in provider code
- broad speculative abstractions for future multi-agent features

If such behavior becomes necessary, first explain **why ACP is insufficient** and **why the logic belongs here rather than upstream or in pi**.

## Layering Rules

### pi owns
- top-level harness behavior
- session UX
- memory / agenda / delegation conventions
- broader agent workflow

### this repository owns
- the narrow bridge from pi provider calls to ACP transport
- bridge-specific visibility and lifecycle control

### claude-agent-acp owns
- Claude-specific ACP server behavior

### Claude Code owns
- Claude-side native runtime behavior
- native config loading
- Claude-side MCP / skills / shell execution

When in doubt, push responsibility **down to the canonical layer** or **up to pi**, not sideways into this repo.

## Non-Append Preference

When a capability is already available through Claude Code’s standard paths (for example `~/.claude`, native MCP config, or shell/PATH tools), prefer using that path over duplicating the same information in the bridge.

This does **not** mean “do nothing.”
It means:

- do not duplicate configuration blindly
- do not rebuild execution paths unnecessarily
- do make execution visible to the pi user
- do keep session state honest when history changes

## Change Strategy

Prefer changes that are:

- small
- explicit
- testable
- easy to delete
- easy to reason about from one file at a time

Avoid changes that are:

- magical
- compensatory
- stateful in multiple layers
- blind to what Claude is actually doing
- hard to validate without reading the entire codebase

## Documentation Rules

- All repository documentation should be written in **English**.
- Keep the README architecture and status sections current.
- If the bridge contract changes, update `README.md` in the same change.
- If the operating model for capability loading changes, update `README.md` and this file together.
- If the development workflow changes, update `run.sh` documentation and examples.

## Validation Commands

Use these before finishing significant changes:

```bash
npm install
npm run typecheck
./run.sh smoke .
```

If the change affects process spawning, prompt flow, session reuse, tool visibility, or bridge settings, run the smoke test again after the final edit.

## Local Files and Hygiene

Do not commit local harness state such as:

- `.pi/settings.json`
- ad-hoc local auth files
- machine-specific temporary debugging artifacts

Keep the repository portable.

## Review Standard

The correct review question for this repository is not:

> “Can we make it do more?”

The correct review questions are:

> “Did this change keep the bridge thin?”
>
> “Did this change improve observability?”
>
> “Did this change preserve a clean boundary between pi and Claude Code?”
