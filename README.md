# abhiabhi94/.github

Account-wide GitHub Actions shared by my repositories.

## Codex PR review (`codex-review-reusable.yml`)

A reusable workflow that has OpenAI Codex review a pull request and post the
result as one PR comment. Codex reads the calling repository's `AGENTS.md`
on its own, so review rules live with each repo (a "Code review" section in
`AGENTS.md` is the place for them) and nothing repo-specific lives here.

### Enable it in a repository

1. Add the `OPENAI_API_KEY` secret to the repo (Settings → Secrets and
   variables → Actions). Personal accounts have no shared secrets, so each
   repo sets its own.
2. Make sure the repo has an `AGENTS.md` at its root. If other agents use
   `CLAUDE.md`, make that a shim whose first line is `@AGENTS.md`.
3. Add `.github/workflows/codex-review.yml`:

```yaml
name: Codex review
on:
  pull_request:
    types: [opened, ready_for_review, labeled]
permissions:
  contents: read
  pull-requests: write
jobs:
  codex:
    uses: abhiabhi94/.github/.github/workflows/codex-review-reusable.yml@main
    secrets: inherit
```

The review runs for non-draft PRs when they are opened or marked ready for
review, and again whenever the `codex-review` label is added (re-add it to
re-review after new pushes; it does not run on every push, to keep API spend
predictable). Optional inputs, passed under `with:`:

| Input                | Default        | Meaning                                              |
|----------------------|----------------|------------------------------------------------------|
| `label`              | `codex-review` | Label that (re)triggers a review                     |
| `model`              | action default | Model override for `openai/codex-action`             |
| `effort`             | action default | Reasoning effort override                            |
| `extra-instructions` | empty          | Extra review instructions appended to the prompt     |

Codex runs in the read-only permission profile, so it can inspect the
checkout but not change it. `openai/codex-action` refuses actors without
write access to the repo, so a pull request from a fork cannot spend the key.
