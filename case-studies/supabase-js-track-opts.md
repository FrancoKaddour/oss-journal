# Case study: shipping a fix to supabase-js

**The problem.** `supabase-js` users configuring Realtime presence could pass options to `channel.track(payload, opts)` — a documented parameter that was silently ignored: `track()` built its message but never forwarded `opts` to the underlying `send()`. Custom timeouts didn't apply, and there was no error to tell you why ([#2466](https://github.com/supabase/supabase-js/issues/2466)).

**The investigation.** The gap was in `RealtimeChannel`: both `track()` and `untrack()` constructed the payload and called `send()` with its default options, dropping the caller's. A one-line-shaped bug — but in a library with hundreds of thousands of dependents, the fix needed to prove it changed exactly one behavior.

**The fix.** Forward `opts` through `track()`/`untrack()` to `send()`, preserving the default when no options are given. When the maintainer asked for test coverage during review, I shipped three tests the same day — asserting the forwarding in both methods and the unchanged default path. The review turnaround mattered: request in the morning, tests pushed by the afternoon, "LGTM thanks!" after.

**The outcome.** Merged in [#2490](https://github.com/supabase/supabase-js/pull/2490) and shipped in [v2.110.6](https://github.com/supabase/supabase-js/releases/tag/v2.110.6), with credit by name in the official release notes. The fix now runs in production for every project on current supabase-js.

**What I'd tell you in an interview.** The code was the easy part. What earned the merge was treating the maintainer's review as the day's top priority and answering with code, not promises.
