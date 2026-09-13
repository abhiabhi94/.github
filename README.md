# abhiabhi94/.github

Account-wide GitHub Actions shared by my repositories.

## AI PR review (`pr-review-reusable.yml`)

A reusable workflow that runs [PR-Agent](https://github.com/The-PR-Agent/pr-agent)
against a pull request through [OpenRouter](https://openrouter.ai/) and posts
the review (plus inline code suggestions) on the PR. PR-Agent reads the
calling repository's `AGENTS.md` on its own, so review rules live with each
repo (a "Code review" section in `AGENTS.md` is the place for them) and
nothing repo-specific lives here.

### Enable it in a repository

1. Add the `OPENROUTER_API_KEY` secret to the repo (Settings → Secrets and
   variables → Actions). Personal accounts have no shared secrets, so each
   repo sets its own. Keys: https://openrouter.ai/settings/keys
2. Make sure the repo has an `AGENTS.md` at its root on the default branch
   (PR-Agent reads it from there, so rule changes apply once merged). If
   other agents use `CLAUDE.md`, make that a shim whose first line is
   `@AGENTS.md`.
3. Add `.github/workflows/pr-review.yml`:

```yaml
name: PR review
on:
  pull_request:
    types: [opened, reopened, ready_for_review]
  issue_comment:
    types: [created]
permissions:
  contents: read
  issues: write
  pull-requests: write
jobs:
  review:
    uses: abhiabhi94/.github/.github/workflows/pr-review-reusable.yml@main
    secrets: inherit
```

### What runs when

- A non-draft PR is opened, reopened, or marked ready for review: `/review`
  and `/improve` run automatically. `/describe` is off by default so your
  PR description is not rewritten.
- A comment starting with `/` on a PR runs that PR-Agent tool on demand:
  `/review`, `/improve`, `/describe`, `/ask <question>`, `/update_changelog`.
  Use `/review` after pushing more commits; the workflow does not run on
  every push, to keep model spend predictable.
- Comments from bots and PRs from forks are skipped (fork PRs get no secrets
  anyway). Do not switch the trigger to `pull_request_target` to "fix" that.

### Inputs (pass under `with:` in the caller)

| Input                     | Default           | Meaning                                                              |
|---------------------------|-------------------|----------------------------------------------------------------------|
| `model`                   | `openrouter/free` | PR-Agent model id (LiteLLM format), e.g. `openrouter/anthropic/claude-sonnet-4.5` |
| `fallback-model`          | same as `model`   | Retry model if the primary call fails                                |
| `custom-model-max-tokens` | `32000`           | Context size assumed for models PR-Agent does not have in its table  |
| `auto-improve`            | `true`            | Post `/improve` suggestions when a PR opens                          |
| `auto-describe`           | `false`           | Rewrite the PR title/description with `/describe` when a PR opens    |
| `extra-instructions`      | empty             | Extra free-text instructions for `/review`, on top of `AGENTS.md`    |

`openrouter/free` routes to whichever free models OpenRouter offers at the
time; quality and rate limits vary, and free routes may use prompts for
provider training under OpenRouter's data policy. Fine for public repos;
switch `model` to a paid one for private code or when reviews miss things.
Changing the model is one input in the caller (or the default here).
