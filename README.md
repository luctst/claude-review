# claude-review

Reusable workflow that runs a fresh Claude agent to review a PR when someone with write access comments `/review` on it.

## Use it in a repo

1. Add an `ANTHROPIC_API_KEY` secret to the repo (or org).
2. Add `.github/workflows/claude-review.yml`:

```yaml
name: Claude review
on:
  issue_comment:
    types: [created]

jobs:
  review:
    uses: luctst/claude-review/.github/workflows/review.yml@main
    secrets: inherit
    # with:
    #   extra_prompt: "This repo is a Swift iOS app; check for main-thread UI updates."
```

Comment `/review` on a PR to run a review. Comment it again for a fresh one.

Each `/review` runs two fresh agents, one after the other, each posting its own review:
1. A general review: bugs, security, breaking changes.
2. An over-engineering review following `prompts/ponytail-review.md` and `prompts/ponytail.md`. It skips what the first review already said.

Edit the prompts here and every repo picks up the change from `main`.

This repo must be public: the workflow checks out `prompts/` with the calling repo's token, which can't read another private repo.