# Case study: four suspects, five probes — a JSDoc parsing bug in Storybook

**The problem.** Storybook's maintainer filed a terse issue: language-tagged code fences (` ```tsx `) in JSDoc `@example` blocks didn't render as code blocks — untagged fences worked. No repro, no diagnosis, and a pipeline with at least four parsing layers that could each be the culprit: `comment-parser`, `react-docgen`, `react-docgen-typescript`, and `markdown-to-jsx`.

**The investigation.** Instead of theorizing, I wrote a small probe for each layer using the exact versions locked in the repo:

- `comment-parser` treats tagged and untagged fences identically — but its `compact` spacing mode collapses multi-line tag values onto one line.
- `react-docgen` keeps descriptions verbatim. Not involved.
- `react-docgen-typescript` splits examples into `tags` with newlines intact. Not involved.
- `markdown-to-jsx` (locked version) renders both fence styles correctly. Not involved.

That left one suspect standing: `extractJSDocInfo` in the React component manifest parses JSDoc tags with `spacing: 'compact'`. A fenced `@example` collapses to a single line — and the two fence styles degrade *differently*, which explained the reported asymmetry exactly: an untagged fence reads as an inline code span with the right content, while a tagged fence leaks its language into the visible text (`tsx <Button />`) and never renders a block.

**The fix.** Surgical: tag values containing a fence fall back to the `preserve` parse the function already runs (it parses twice by design). Values without fences keep the single-line shape existing consumers and snapshots expect. Red-green verified — the new fence tests fail against the previous implementation — and the package's full suite (278 tests) stays green.

**The outcome.** [PR #35527](https://github.com/storybookjs/storybook/pull/35527) in review, clean automated review.

**The takeaway.** With four suspects and no repro, the difference between guessing and knowing was five 20-line scripts. Empirical elimination beats plausible theories — every time.
