# CI Notes

Notes on the CI setup in this repo and on running Claude Code as a CI worker.
See also: [Claude Code as a CI Worker](crash-courses/13-claude-code-ci-crash-course.md).

## Workflows in this repo

| Workflow | File | Trigger | Purpose |
|---|---|---|---|
| Claude Code | `.github/workflows/claude.yml` | `@claude` in an issue, issue comment, PR comment, or PR review | Claude works on the request, can push commits and open PRs |
| Claude Code Review | `.github/workflows/claude-code-review.yml` | PR opened, synchronized, reopened, or ready for review | Automatic code review posted as inline PR comments |
| markdown-only | `.github/workflows/markdown-only.yml` | PR targeting `main` | Fails the PR if it adds or changes non-Markdown files |

## How the pieces work

- **`@claude` mentions**: `claude.yml` only runs its job when the event text contains `@claude`, so ordinary activity costs nothing.
- **Auth**: the action reads `CLAUDE_CODE_OAUTH_TOKEN` from repository secrets. Never hard-code tokens in workflow files.
- **Permissions**: jobs request the minimum (`contents`, `pull-requests`, `issues` read; `id-token: write` for the action's auth; `actions: read` so Claude can read CI results on PRs).
- **Review job**: uses the `code-review` plugin and is limited to the inline comment tool through `--allowedTools`.
- **Guardrail**: `markdown-only.yml` diffs against `origin/<base>...HEAD` with `--diff-filter=ACMR` (deletions are allowed) and allows only `*.md`, `.github/`, `.gitignore`, and `LICENSE`.

## Limitations to remember

- Claude's GitHub App cannot modify files under `.github/workflows/`. Workflow changes must be made by a human.
- Claude updates a single comment per run rather than posting new ones.
- Claude cannot approve PRs or submit formal reviews.
- Claude has no memory between runs. Put durable instructions in a `CLAUDE.md`.

## Good practices

1. Keep the trigger narrow (`@claude` only) and the permissions minimal.
2. Treat Claude's PRs like any other: require a human review before merging.
3. Keep fast, deterministic checks (like `markdown-only`) alongside AI review. The AI reviewer is not a substitute for them.
4. Pin action versions and review changes to workflow files carefully.
5. Watch usage: the review job runs on every PR update.
