# AI Help Coverage

Fails CI when a **visible** (non-`Hidden`) JFrog CLI command ships without
AI-oriented help, so agents never silently fall back to the short human
description. No code is committed to the consuming repos — the check lives
entirely in this action.

## How it works

1. **Detect** — discovers the command entry point in the repo (no per-repo config):
   - a **plugin** registers commands through one or more functions with the
     signature `func Xxx() components.App` — the action finds them by signature and
     checks every one (so a repo exposing several Apps is fully covered);
   - the **umbrella** (`package main`) assembles the tree via `getCommands()`.
   A repo with neither is a safe no-op.
2. **Generate** — writes a temporary `*_test.go` into the target package. The test
   builds the command tree twice — with `JFROG_CLI_AI_HELP=false` and `=true` — and
   collects the rendered help (`Usage`) for every visible leaf command.
3. **Check** — `go test` runs the generated test. A command whose help is identical
   in both modes has no AI help (its `AIDescription` is empty or not routed through
   `ResolveDescription`) and fails the check.
4. **Cleanup** — the generated file is always removed (`if: always()`), pass or fail.

Because it diffs the *rendered* output, it also catches the case where an
`AIDescription`/`GetAIDescription()` exists but the call site forgot to resolve it.

In the umbrella (`jfrog-cli`) the check builds the full command tree but verifies
only the umbrella's **own** commands — commands contributed by embedded plugins are
excluded, since each plugin is checked in its own repo.

## Usage

Add a step to a job that has already set up Go (e.g. the Static Analysis workflow):

```yaml
- uses: jfrog/.github/actions/install-go-with-cache@main
- uses: jfrog/.github/actions/ai-help-coverage@main
```

## Inputs

| Input | Default | Description |
| --- | --- | --- |
| `skip-commands` | `''` | Comma/newline-separated command paths to exclude. A path matches itself and its whole subtree (e.g. `agent skills` drops `agent skills install`). Empty = check everything. |
| `generated-file-name` | `zzz_aihelp_coverage_generated_test.go` | Name of the temporary test file written and then deleted. |

```yaml
- uses: jfrog/.github/actions/ai-help-coverage@main
  with:
    skip-commands: |
      agent skills
      agent plugins
```

## Supported repos

Any JFrog CLI repo is detected automatically:

- **plugins** — any repo exposing one or more `func Xxx() components.App`
  (`jfrog-cli-security`, `jfrog-cli-artifactory`, `jfrog-cli-evidence`,
  `jfrog-cli-platform-services`, `jfrog-cli-application`, and any future plugin);
- **umbrella** — `jfrog-cli`, where only the umbrella's own commands are checked
  (commands contributed by embedded plugins are excluded).

A new plugin needs no change here — as long as its App accessor follows the
`func Xxx() components.App` signature, it is picked up automatically.

## Fixing a failure

For each reported command, add a `GetAIDescription()` to its `docs/.../help.go` and
resolve it via `corecommon.ResolveDescription(GetDescription(), GetAIDescription())`
at the command's call site — or mark the command `Hidden` if it is not for agents.
