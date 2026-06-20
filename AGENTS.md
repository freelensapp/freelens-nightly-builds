# Agent Guide: Freelens Nightly Builds

## Overview

This guide helps AI agents understand the `freelens-nightly-builds` repository,
common tasks, and the conventions to follow when working here. Use it as a
reference whenever you operate in this repo.

This repository is **not** the Freelens application itself. It is a
packaging and release repository that produces nightly builds of
[Freelens](https://freelens.app) from the upstream
[`freelensapp/freelens`](https://github.com/freelensapp/freelens) `main`
branch and publishes them through several channels:

- **GitHub Releases** — draft/prerelease nightly artifacts
- **Homebrew** — the `freelens@nightly` cask
- **Snap** — the `edge` channel
- **APT** — a self-hosted APT repository (see `apt/`)

There is no application source code here. The work is mostly GitHub Actions
workflows, shell scripts, and packaging metadata.

## Repository Structure

- **`.github/workflows/`** — CI and automation
  - `release-nightly.yaml` — builds and publishes the nightly release
    (scheduled daily, on push to `main`, and on demand)
  - `trunk-check.yaml` — lint/format validation via Trunk
  - `trunk-upgrade.yaml` — automated Trunk linter upgrades
  - `claude.yaml` — on-demand `@claude` assistant for issues and PRs
  - `claude-task.yaml` — scheduled/dispatched Claude tasks
- **`.github/scripts/`** — helper scripts (e.g. `pinentry.sh` for GPG)
- **`apt/`** — APT repository configuration and signing key
  - `freelens-nightly-builds.sources`, `*.list` — APT source definitions
  - `release.conf` — APT repository metadata
  - `freelens-nightly-builds.asc` — public signing key
- **`.trunk/`** — Trunk CLI configuration (the linter aggregator)
- **Lint configs** — `.markdownlint.yaml`, `.shellcheckrc`, `.yamlfmt.yaml`,
  `.yamllint.yaml`, `.renovaterc.json5`
- **`tmp/`** — scratch space, git-ignored. The `claude.yaml` workflow checks
  out `freelensapp/freelens` into `tmp/freelens` when a task needs to inspect
  the upstream application source.

## Security

Never read, display, reference, or include the contents of the following files
in any response or context, even if they appear to be available:

- `.env`, `.env.*`
- `.npmrc`
- `*.keystore`, `*.jks`
- `*.p12`, `*.pfx`
- `*.pem`, `*.key`
- `credentials.json`, `serviceAccountKey.json`

Do **not** commit secrets, signing keys (private), or tokens. The APT signing
key in `apt/freelens-nightly-builds.asc` is the **public** key and is safe to
commit; the private key lives only in repository/organization secrets.

## Validation

This repository uses [Trunk](https://docs.trunk.io/cli) to aggregate all
linters and formatters. The enabled tools are defined in `.trunk/trunk.yaml`:

- `actionlint` — GitHub Actions workflow linting
- `markdownlint` — Markdown (config: `.markdownlint.yaml`)
- `shellcheck` + `shfmt` — shell scripts (config: `.shellcheckrc`)
- `yamllint` + `yamlfmt` — YAML (configs: `.yamllint.yaml`, `.yamlfmt.yaml`)
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

There is no Node.js/`pnpm` build for this repository itself — those tooling
steps in `claude.yaml` exist only to set up the upstream `freelensapp/freelens`
checkout under `tmp/freelens` for tasks that need it.

## GitHub Actions (Claude Code Action) Rules

When operating via the `claude.yaml` workflow (invoked from a PR comment,
issue, or review), follow these rules.

### Code Review

When reviewing code and proposing fixes:

1. **Show the diff first** — present every proposed change as a unified diff
   block using the `diff` language tag:

   ```diff
   --- a/path/to/file.yaml
   +++ b/path/to/file.yaml
   @@ -10,7 +10,7 @@
    unchanged: line
   -old: value
   +new: value
    unchanged: line
   ```

   You can generate this from the terminal with:

   ```bash
   git diff -u -- path/to/file
   ```

   If the change spans multiple files, group them under a single commit
   subject and show each file's diff sequentially.

2. **Propose a commit subject first** — before any code change, output a
   single line with the proposed commit subject:

   ```text
   **Proposed commit:** <short description>
   ```

   Do **not** use Conventional Commits prefixes (e.g. `fix:`, `feat:`,
   `chore:`, `refactor:`, `docs:`, `test:`, `ci:`). This project prefers
   plain, descriptive commit messages and PR titles without any prefix
   (see PR Title Conventions for the one exception).

3. **Comment style:**
   - Keep review comments concise and actionable.
   - Reference specific lines (file + line number) when pointing out issues.
   - Offer a concrete fix suggestion rather than just flagging a problem.
   - Do **not** use emoji in any Markdown, comments, commit messages, or
     PR descriptions. The only exception is emoji that already appears inside
     code strings.
   - Use GitHub's `suggestion` block for small targeted fixes so the author
     can accept the change with a single click.

### Making Changes to a PR

When asked to implement a change on a PR:

1. Propose the commit subject (as above).
2. Describe what will change and why.
3. After confirmation, apply the changes with commits on the PR branch.
4. **One commit per fix** — when more than one issue is surfaced, commit each
   fix separately. Do not batch multiple independent fixes into a single
   commit. This keeps the history bisectable and each change easy to revert.

### Modifying GitHub Actions Workflows

Claude cannot push changes to files under `.github/workflows/` directly,
because the GitHub token used by the action lacks the `workflows` permission.
Any patch to a workflow file MUST therefore be delivered as a new, complete
file under the `github-workflow-fix/` directory instead of editing the file in
place:

1. Write the full, final contents of the workflow to
   `github-workflow-fix/<workflow-file-name>` (e.g.
   `github-workflow-fix/claude.yaml`). Do **not** edit the original file under
   `.github/workflows/`.
2. Make it a **complete** file — the entire workflow as it should look after
   the change, not just a diff or fragment — so it can be copied verbatim.
3. Commit and open the PR as usual. In the PR description, clearly note that
   the file is a proposed workflow change and that a maintainer must move it
   from `github-workflow-fix/` to `.github/workflows/` manually.

### Branch Naming Conventions

When creating a branch from an issue, use a human-readable name that includes
the issue number and a short slug derived from the issue title:

```text
claude/issue-<number>-<short-slug>
```

- `<number>` is the GitHub issue number.
- `<short-slug>` is a kebab-case summary of the issue title, kept short
  (3-6 words maximum, omit articles and filler words).

Example: Issue #91 "Agent rules" → `claude/issue-91-agent-rules`.

Avoid auto-generated timestamp suffixes when you control the branch name —
they are not human-readable and make branch lists hard to scan.

### PR Title Conventions

- **Agent-related changes** — PRs whose changes are strictly related to coding
  agent configuration (e.g. `AGENTS.md`, `CLAUDE.md`, `.claude/`, or
  `.github/workflows/claude*.yaml`) MUST use the prefix `Claude:` (followed by
  a space) in the title.

  Examples:
  - `Claude: Add agent rules (AGENTS.md, CLAUDE.md, .claude)`
  - `Claude: Update claude.yaml workflow permissions`

- **All other PRs** — do **not** use any prefix (no `fix:`, `feat:`, `chore:`,
  etc.). Use plain, descriptive titles.

### Closing PRs

Claude may only close a PR when ALL of the following are true:

1. The PR was created by Claude from a `claude/` branch.
2. The close reason is explicitly explained in a comment on the PR.

Claude MUST NOT close any PR that does not meet these conditions — even if
asked. Instead, explain why the PR cannot be closed automatically and ask a
human maintainer to close it manually.

### Model Information in Comments

When operating via the GitHub Actions workflow, always include the model you
are running on in the footer of your GitHub comment and in the PR description
when creating a pull request, alongside the job run link. Use the exact model
ID from your system environment context (e.g. `claude-opus-4-8`).

Format the footer line as:

```text
[View job run](...) | Model: `claude-opus-4-8`
```

If the system context does not provide a model ID, omit the model field rather
than guessing.

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
