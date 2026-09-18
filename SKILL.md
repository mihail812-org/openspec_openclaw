---
name: openspec
description: "Spec-driven development with the openspec CLI (`@fission-ai/openspec`): propose, validate, implement, and archive non-trivial changes. Use when the user says \"propose a change\", \"spec this out\", \"open a change for X\", wants a multi-step feature planned before coding, or asks to close out an existing `openspec/` change."
---

# OpenSpec

Use OpenSpec to align on **what** before writing **how**. All state lives under
`openspec/` at the project root; every in-flight change is its own folder with
`proposal.md`, `specs/`, `design.md`, `tasks.md`.

Drive everything through the `openspec` CLI. Never hand-author the folder
structure or edit `openspec/specs/` mid-change — `archive` is what rolls deltas
into the main spec set.

## Environment

The CLI is preinstalled at `~/.nvm/versions/node/v24.20.0/bin/openspec` (v1.13+).
It resolves on `PATH` for non-login shells; a *login* shell (`sh -lc`, `bash -lc`)
resets `PATH` via `/etc/profile` and loses it. Prefer plain `openspec ...` — if it
reports not found, export the nvm bin dir, then continue.

Set `OPENSPEC_TELEMETRY=0` on first use to silence the anonymous-stats notice.
Always pass `--json` when a command offers it; that is the stable interface for
agent use. Commands resolve the nearest `openspec/` root, so `cd` into the project
first. When a standalone store is named, run `openspec store list --json` once and
pass `--store <id>` on every subsequent command.

## When to use

- The user asks to propose, scope, or specify a change before coding.
- The project already has `openspec/` — keep using it.
- The change spans several files or modules and benefits from a checklist.
- The user wants to close out a completed change.

Skip it for trivial edits (one-line fixes, typos, log tweaks). The ceremony is
not worth it.

## Workflow

1. **Check the project.** `openspec list --json` and `openspec list --specs --json`
   report the active changes and existing capabilities — without `openspec/`,
   they error. To bootstrap, run `openspec init --tools <ids> --profile core`
   (`--tools none` when no assistant files are wanted); it writes
   `openspec/config.yaml`, `openspec/specs/`, and `openspec/changes/archive/`.
   **Done when** `openspec list --json` returns a root.

2. **Open the change.** `openspec new change <kebab-name>` creates
   `openspec/changes/<name>/` with a `.openspec.yaml` pinning the schema.
   **Done when** the command prints the change path.

3. **Write artifacts in dependency order.** `openspec status --change <name>`
   prints `Progress: n/4` and which artifacts are still blocked. For each one, get
   the schema's own brief and write to the file it names:

   ```bash
   openspec instructions <artifact> --change <name> --json
   ```

   The response gives `outputPath` (relative to the change dir) and a full
   `instruction`. Artifacts and their dependencies:

   | Artifact | File | Blocked by |
   | --- | --- | --- |
   | proposal | `proposal.md` | — |
   | specs | `specs/<capability>/spec.md` | proposal |
   | design | `design.md` | proposal |
   | tasks | `tasks.md` | specs, design |

   **Done when** `openspec status --change <name>` reports `4/4 artifacts complete`.

4. **Validate.** `openspec validate <name> --strict --json`; fix every reported
   issue before claiming the change is ready. **Done when** the JSON shows
   `"valid": true`.

5. **Implement.** Work `tasks.md` one item at a time, flipping `- [ ]` to `- [x]`
   as each item actually lands. Keep diffs scoped to the current task.
   **Done when** every box is ticked or the user accepts a partial state.

6. **Archive.** `openspec archive <name> --yes --json`. It merges the delta specs
   into `openspec/specs/` and moves the change to
   `openspec/changes/archive/<YYYY-MM-DD>-<name>/`. Use `--skip-specs` for
   infra/docs-only changes. **Done when** the JSON names `archivedAs`.

## Spec format (the part that silently fails)

Delta specs are matched by header shape; a wrong heading level is accepted but
produces no delta.

- Delta sections: `## ADDED Requirements`, `## MODIFIED Requirements`,
  `## REMOVED Requirements`, `## RENAMED Requirements`.
- Each requirement: `### Requirement: <name>`, described with **SHALL**/**MUST**
  (never should/may).
- Each scenario: `#### Scenario: <name>` — **exactly four hashes** — with
  `- **WHEN** ...` / `- **THEN** ...` lines. Every requirement needs at least one.
- New capabilities start with a `## Purpose` section of 50+ characters; do not add
  `## Purpose` to a delta for an existing capability.
- `MODIFIED` must carry the complete updated requirement block — a partial copy
  loses detail at archive time. Adding behavior without changing existing behavior
  is `ADDED`, not `MODIFIED`.
- Paths must match the project's existing layout; confirm with
  `openspec list --specs --json` before writing a delta against an existing
  capability, and never rename or move a capability.

## Anti-patterns

- Do not invent specs for changes the user did not ask to formalize. OpenSpec is
  for spec-driven flow, not a tax on every edit.
- Do not skip `openspec validate` before declaring a change ready.
- Do not archive with unchecked tasks. The CLI does **not** stop you — verified:
  `archive --yes` succeeds with `- [ ]` items outstanding — so check `tasks.md`
  yourself and get explicit approval for a partial state.
- Do not edit `openspec/specs/` directly during a change; put the delta in the
  change folder and let `archive` roll it in. The one exception is fixing an
  existing capability's `## Purpose`.
- Do not hand-create `openspec/` files when a subcommand or `openspec instructions`
  already owns that step.

For worked scenarios, see `references/workflow.md`. Other useful reads:
`openspec templates --json`, `openspec schemas --json`, `openspec context --json`,
`openspec doctor --json`.
