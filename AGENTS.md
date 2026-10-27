# Agent Guide: Freelens Nightly Builds

## Overview

This guide helps AI agents understand the `freelens-nightly-builds` repository,
common tasks, and the conventions to follow when working here. Use it as a
reference whenever you operate in this repo.

This repository is **not** the Freelens application itself. It is a
packaging and release repository that produces nightly builds of
[Freelens](https://freelens.app) from the upstream
[`freelensapp/freelens`](https://github.com/freelensapp/freelens) `main`
branch and publishes them as GitHub releases. Other channels are fed from
those releases:

- **GitHub Releases** — one release per day, tagged
  `v<version>-nightly-YYYY-MM-DD`. It is created as a draft, filled by the
  build jobs and then published as a regular release, not a prerelease, so
  `releases/latest` points at it. A run for a version whose release is
  already published skips the build and leaves that release as it is; a draft
  left by a failed run is replaced. Only the last ten releases are kept.
- **APT** — signed repository index files (`Packages`, `Release`,
  `InRelease`) uploaded to each release next to the `.deb` files, served from
  `releases/latest/download` (see `apt/`). They are made by
  `release-apt-repository.yaml` after the release is published, so until that
  run finishes the latest release has no index files.
- **Homebrew** — the `freelens@nightly` cask, which lives in upstream
  [`Homebrew/homebrew-cask`](https://github.com/Homebrew/homebrew-cask) and
  is bumped there, not from this repository
- **Snap** — the `edge` channel, published by the separate
  [`freelensapp/freelens-snap`](https://github.com/freelensapp/freelens-snap)
  repository from the `.deb` files of the latest release
- **npm** — `@freelensapp/extensions` under the `nightly` dist-tag, published
  from today's published nightly release by a manual run of `publish-npm.yaml`. It
  is opt-in because npm has no counterpart to the cleanup that keeps only the
  last ten GitHub releases: every published version stays in the registry for
  good. The npm version is the same as the binary packages'
  (`<version>-nightly-YYYY-MM-DD`), which sorts correctly only because upstream
  uses no prerelease other than `-0`: a numeric prerelease such as `2.0.0-1`
  would sort below any nightly, and `-10-nightly-…` below `-9-nightly-…`

There is no application source code here. The work is mostly GitHub Actions
workflows, shell scripts, and packaging metadata.

## Repository Structure

- **`.github/workflows/`** — CI and automation
  - `release-nightly.yaml` — builds and publishes the nightly release
    (scheduled daily at 02:00 UTC and on demand). A pull request to `main`
    that changes this file runs the build without publishing anything. A push
    to `main` does not trigger it.
  - `release-apt-repository.yaml` — builds, signs and uploads the APT
    repository index files to the latest release after every successful run
    of `release-nightly.yaml` on `main` (unless that release already has
    them), and by hand to any release, without rebuilding it. It runs on
    `main` only, because the `apt-signing` environment that holds the signing
    key allows no other branch.
  - `publish-npm.yaml` — publishes `@freelensapp/extensions` from today's
    nightly release to npm (manual only, fails if that GitHub release is
    missing or still a draft)
  - `trunk-check.yaml` — lint/format validation via Trunk (on push to any
    branch except `automated/*` and `renovate/*`, and on pull requests to
    `main`)
  - `trunk-upgrade.yaml` — automated Trunk linter upgrades, opened as a pull
    request from the `automated/trunk-upgrade` branch (daily, on push to
    `main` that changes `.trunk/`, and on a `/rerun` comment on that pull
    request)
  - `biome-migrate.yaml` — on a Renovate pull request that changes
    `.trunk/trunk.yaml` or `biome.jsonc`, runs `biome migrate` for the Biome
    version pinned in `.trunk/trunk.yaml` and commits the migrated
    `biome.jsonc` to the same branch
  - `mise-lock-check.yaml` — on pull requests that change `mise.toml` or
    `mise.lock`, installs the tools from the lock file and fails when a
    checksum changed while the tool version did not
  - `claude.yaml` — on-demand `@claude` assistant for issues and PRs (a PR
    from a fork gets a review only)
  - `claude-task.yaml` — a task for Claude started by hand, with the model
    and the runner to use (manual only)
- **`.github/claude/ci-agent.md`** — the rules for the agent running in the
  Claude workflows (see "GitHub Actions (Claude Code Action) Rules")
- **`.github/scripts/`** — helper scripts (e.g. `pinentry.sh` for GPG)
- **`apt/`** — APT repository configuration and signing key
  - `freelens-nightly-builds.sources`, `*.list` — APT source definitions
  - `release.conf` — APT repository metadata
  - `freelens-nightly-builds.asc` — public signing key
- **`.trunk/`** — Trunk CLI configuration (the linter aggregator)
- **Lint configs** — `biome.jsonc`, `.shellcheckrc`, `.yamllint.yaml`,
  `.renovaterc.json`
- **`tmp/`** — scratch space, git-ignored. The `claude.yaml` workflow checks
  out the `main` branch of `freelensapp/freelens` into `tmp/freelens`, so a
  run can inspect the upstream application source.

## Security

Never read, display, reference, or include the contents of the following files
in any response or context, even if they appear to be available:

- `.env`, `.env.*`, `.envrc`
- `.npmrc`
- `*.keystore`, `*.jks`
- `*.p12`, `*.pfx`
- `*.pem`, `*.key`

The same list is git-ignored in `.gitignore` and enforced for Claude Code by
the `permissions.deny` rules in `.claude/settings.json`, which block reading
and editing these files. Change all three together. The rules are native
permissions rather than a hook on purpose: a hook runs a process in the
working tree, which may be an untrusted pull request, and an interpreter such
as `python3 -c` imports modules from that tree before the hook's own code.

Do **not** commit secrets, signing keys (private), or tokens. The APT signing
key in `apt/freelens-nightly-builds.asc` is the **public** key and is safe to
commit; the private key lives only in repository/organization secrets.

## Validation

This repository uses [Trunk](https://docs.trunk.io/cli) to aggregate all
linters and formatters. The enabled tools are defined in `.trunk/trunk.yaml`:

- `actionlint` — GitHub Actions workflow linting, from the maintained fork
  `kjanat/actionlint` (see `downloads` in `.trunk/trunk.yaml`)
- `biome` — formats JSON, Markdown and YAML, and lints Markdown and YAML
  (config: `biome.jsonc`), the same as upstream `freelensapp/freelens`
- `shellcheck` + `shfmt` — shell scripts (config: `.shellcheckrc`)
- `yamllint` — YAML (config: `.yamllint.yaml`), only for the rules Biome
  does not have yet: `quoted-strings` and `octal-values`. Biome never adds
  or removes quotes, so the two do not conflict. A YAML rule that Biome
  gains goes to `biome.jsonc` (as `key-duplicates` of yamllint did, now
  `noDuplicateMapKeys`), and yamllint goes once Biome covers the rest.
- `trufflehog` — secret scanning
- `git-diff-check` — whitespace/conflict-marker checks

Before committing any change, validate it the same way CI does:

```bash
trunk check            # lint changed files (or `trunk check --all`)
trunk fmt              # auto-format changed files
```

If the `trunk` CLI is not installed locally, the `trunk-check.yaml` workflow
runs the same checks on push and pull request. Fix all reported issues before
considering a change complete.

There is no Node.js/`pnpm` build for this repository itself. After changing a
version in `mise.toml`, regenerate the lock with `mise lock`, which refreshes
every platform in the file; `mise install` re-locks only the platform it runs
on.

The tools that Freelens also has follow the Freelens stack: Biome and the Node
runtime in `.trunk/trunk.yaml`, and the tools in `mise.toml`. Custom
datasources in `.renovaterc.json` read their versions from the root
`package.json` (the `pnpm dlx` scripts) and `mise.toml` of `freelensapp/freelens`
on `main`, so an update arrives only once Freelens has adopted it, at that
version, in one `Freelens dependencies` pull request. A tool added to
`mise.toml` that Freelens does not have fails its lookup on the Dependency
Dashboard and needs a rule of its own. Everything else (GitHub Actions, the
tool versions in the workflows, the Trunk linters) Renovate and
`trunk-upgrade.yaml` update as usual.

## Local Agent: Triggering the GitHub Agent

These rules apply to an agent running on a developer machine (a local Claude
Code session), not to the workflow agent. The local agent shares the repository
with the CI agent defined in `.github/workflows/claude.yaml`, and every comment
it writes on GitHub is a potential trigger for it.

### How the trigger works

`claude.yaml` starts a run when the body of a **newly created** comment (issue
comment or PR review comment), a **newly opened** issue (body or title), or a
**submitted** PR review contains the string `@claude`, and its author has
write access to the repository (the admin, maintain or write role). The
workflow condition pre-filters on OWNER, MEMBER or COLLABORATOR, which does not
imply write access, and the first step of the run checks the actual
permission; a run triggered by anyone else fails there. The check is a plain
`contains(github.event.comment.body, '@claude')` substring test, so the string
fires the workflow wherever it appears — including inside a code span, a fenced
block, a quoted line, or a URL. Markdown formatting is not an escape.

The trigger text may also carry `[model:<alias>]`, `[effort:<level>]` and
`[runs-on:<alias>]` markers, which select the model, the reasoning effort and
the runner for that run (see the `parse` job for the accepted aliases). They are
only read from the triggering text.

The default model is `claude-opus-5-5[1m]` (Opus 5.5 with the 1M-token
context) and it runs at `high` effort. Naming a model explicitly drops that
default: the run then uses the CLI default effort unless `[effort:...]` also
says otherwise. Accepted levels are `low`, `medium`, `high`, `xhigh` and
`max`; anything else is ignored with a note in the job log.

### Rules for the local agent

1. **Write the handle only to start a run.** Ask the user before triggering: a
   run is a 120-minute CI job on the repository, so it is the user's call, not
   an implementation detail.
2. **Escape the handle when merely referring to it.** In issue bodies, PR
   descriptions, review notes, commit messages and documentation, write
   `@<!-- -->claude` (displays as the handle, but the raw body does not contain
   the literal string, so `contains()` does not match) or describe it in prose
   as "the Claude handle". This is what keeps a plan or a bug report that
   documents the trigger from firing it.
3. **Editing never triggers.** The workflow subscribes only to `created`,
   `opened` and `submitted` events — not `edited`. So updating a comment, an
   issue body or a PR description is always safe, even when the text already
   contains a real trigger, and conversely editing a comment to add the handle
   does **not** start a run: a new comment is required.
4. **One trigger per task.** Do not repeat the handle in follow-up comments
   while a run is in flight; each occurrence starts another concurrent job.
5. **Push first.** The workflow checks out the remote ref (the PR head, or the
   default branch for issues), so anything not pushed is invisible to it.

## GitHub Actions (Claude Code Action) Rules

The rules for the agent running in the workflows live in
[`.github/claude/ci-agent.md`](.github/claude/ci-agent.md), and reach that
agent through its system prompt: the workflows fetch the file from `main`,
next to this one. No local session loads it, so change those rules there.

### Modifying GitHub Actions Workflows

The agent running in the workflows cannot push changes under
`.github/workflows/`, because its GitHub token lacks the `workflows`
permission. It delivers a workflow change as a complete file under
`github-workflow-fix/` instead, for a maintainer to move into place. The full
rule is
["Modifying GitHub Actions Workflows"](.github/claude/ci-agent.md#modifying-github-actions-workflows)
in `.github/claude/ci-agent.md`.

## Best Practices

1. **Follow existing patterns** — grep for similar shell/YAML constructs before
   inventing new ones.
2. **Validate before committing** — run `trunk check` (and `trunk fmt`) on every
   change.
3. **Keep workflows minimal** — this repo orchestrates packaging; avoid adding
   build complexity that belongs upstream in `freelensapp/freelens`.
4. **No emoji** in Markdown, comments, commit messages, or PR descriptions
   (except inside existing code strings).
5. **Do not use Anthropic Fable for coding tasks** — Fable may be used only for
   planning, analysis, and reasoning. When writing or editing files, use the
   standard editing tools.

## Getting Help

- Read the workflow files in `.github/workflows/` to understand how releases
  and automation are wired.
- Consult the upstream [`freelensapp/freelens`](https://github.com/freelensapp/freelens)
  repository for anything about the application itself.
- Review the repository's PR and commit history for related changes.
