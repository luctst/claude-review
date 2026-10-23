# claude-review

Reusable workflow that runs a fresh Claude agent to review a PR when someone with write access comments `/review` on it.

## Use it in a repo

1. Run `claude setup-token` locally (needs a Claude Pro/Max account) and add the token as a `CLAUDE_CODE_OAUTH_TOKEN` secret to the repo:
   `gh secret set CLAUDE_CODE_OAUTH_TOKEN --repo <owner>/<repo>`
2. Add `.github/workflows/claude-review.yml`:

```yaml
name: Claude review
on:
  issue_comment:
    types: [created]

jobs:
  review:
    permissions:
      contents: read
      pull-requests: write
      issues: write
      id-token: write
    uses: luctst/claude-review/.github/workflows/review.yml@main
    secrets:
      CLAUDE_CODE_OAUTH_TOKEN: ${{ secrets.CLAUDE_CODE_OAUTH_TOKEN }}
    # with:
    #   extra_prompt: "This repo is a Swift iOS app; check for main-thread UI updates."
```

Pass only this secret, not `secrets: inherit`: this workflow runs from `main` of this repo, and inherit would give it every secret of your repo. The `permissions` block is needed when your repo's default workflow token is read-only.

Comment `/review` on a PR to run a review. Comment it again for a fresh one.

Each `/review` runs two fresh agents, one after the other, and posts one review:
1. A general review: bugs, security, breaking changes. It posts nothing and hands its findings to the second agent.
2. An over-engineering review following `prompts/ponytail-review.md` and `prompts/ponytail.md`. It skips what the first review already said, then posts one review with both: a summary plus inline comments.

The review requests changes when either agent finds a blocking issue, and approves otherwise.

Edit the prompts here and every repo picks up the change from `main`.

This repo must be public: the workflow checks out `prompts/` with the calling repo's token, which can't read another private repo.