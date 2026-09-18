# OpenSpec workflow patterns

Every example assumes you `cd` into the project and have exported
`OPENSPEC_TELEMETRY=0`. Add `--json` to any read command when you need to parse
the result.

## Starting a new change

User: "Let's add dark mode to the dashboard."

1. `openspec list --json` — confirm nothing is already in flight under that name.
2. `openspec new change add-dark-mode` — creates `openspec/changes/add-dark-mode/`.
3. Build the artifacts in dependency order, re-reading the brief each time:

   ```bash
   openspec instructions proposal --change add-dark-mode --json
   openspec status --change add-dark-mode
   ```

   `proposal.md` lists the capabilities; that list decides the spec paths, e.g.
   `specs/theme-preference/spec.md`.
4. `openspec validate add-dark-mode --strict --json`.
5. `openspec status --change add-dark-mode` — expect `4/4 artifacts complete`.

## Picking up an in-flight change

User: "Continue the auth refactor."

1. `openspec list --json` — get the exact change name.
2. `openspec show <name> --json` — read the change; `--diff` shows
   per-requirement deltas, `--deltas-only` only the deltas.
3. `openspec status --change <name> --json` — which artifacts are still missing.
4. Open `openspec/changes/<name>/tasks.md` and resume at the first unchecked box.

Writing the artifact file directly is the intended path — `openspec instructions
<artifact> --change <name> --json` returns the `resolvedOutputPath` to write to.
There is no CLI command that authors artifact content for you.

## Closing out a change

User: "We shipped dark mode."

1. Confirm every box in `openspec/changes/<name>/tasks.md` is `[x]`. **The CLI will
   not check this for you** — `openspec archive <name> --yes` succeeds with
   unchecked tasks still present.
2. `openspec validate <name> --strict --json`.
3. `openspec archive <name> --yes --json` — merges deltas into `openspec/specs/`
   and moves the change to `openspec/changes/archive/<YYYY-MM-DD>-<name>/`.
   The JSON reports `archivedAs` plus added/modified/removed/renamed counts.
4. `openspec list --specs --json` — confirm the merged capability is present.

Use `--skip-specs` when the change is infrastructure, tooling, or docs only and
carries no spec-level behavior change.

## Bootstrapping a project

User: "Set up OpenSpec on this repo."

1. Ask which AI assistant(s) the project targets, then map them to `--tools` ids
   (`claude`, `cursor`, `codex`, `opencode`, `zed`, `windsurf`→`devin`, …); `--tools
   all` or `--tools none` are also accepted.
2. `openspec init --tools <ids> --profile core`.
3. Fill in `openspec/config.yaml` — the optional `context:` block is shown to the
   agent when it writes artifacts, and `rules:` can pin per-artifact constraints.
4. After upgrading the package, run `openspec update` to regenerate the assistant
   instruction files.

## Validation beyond a single change

```bash
openspec validate --all --json          # every change and spec
openspec validate --changes --json      # all changes
openspec validate --specs --json        # all specs
openspec validate --archived --json     # archived changes must have all tasks done
openspec validate <name> --strict --json
```

`--report findings` (with `--json`) trims a bulk report to problems only. Bulk
runs default to 6 concurrent validations; override with `--concurrency <n>` or
`OPENSPEC_CONCURRENCY`.

## Diagnosing an unexpected result

- `openspec doctor --json` — relationship health for the resolved root.
- `openspec context --json` — the working context/brief for that root.
- `openspec schemas --json` — available workflows (`spec-driven` is the default:
  proposal → specs → design → tasks). `openspec schema fork` / `openspec schema
  init` build custom sequences.
- `openspec templates --json` — where each artifact template resolves from
  (`package` vs project override).
- `openspec store list --json` — registered standalone stores, when the change
  does not live in the current repo.

## Pairing with other workflows

- With TaskFlow: a long-running detached implementation can be modelled as a
  TaskFlow whose owner conversation manages the change folder.
- Keep semantic code exploration in whatever code-understanding tools are
  available; OpenSpec covers planning and the artifact lifecycle only.
