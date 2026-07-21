# Case study: the stack trace disagreed with the bug report

**The problem.** Storybook 10.5 users hit an uncaught `TypeError: Illegal invocation` on the first load of a Docs page. The reporter came with an elaborate theory: cross-realm iframes and mounting race conditions. A second issue reported Zag's focus-visible breaking in browser tests after the same upgrade — seemingly unrelated.

**The investigation.** The stack trace told a different story than the report: `HTMLElement.get [as focus]` — a *getter*, living in Storybook's own preview runtime. Storybook 10.5's focus instrumentation replaces `HTMLElement.prototype.focus` with an accessor whose getter starts by touching `this.ownerDocument`.

That's safe for instance calls. But focus-management libraries — react-aria (bundled in addon-docs), Zag — capture the method by *reading it off the prototype* before wrapping it:

```js
const focus = HTMLElement.prototype.focus; // getter runs with `this = HTMLElement.prototype`
```

With the prototype as receiver, the native `ownerDocument` accessor brand-checks and throws. One root cause, two bug reports: in browsers it throws (the Docs-page crash); in DOM environments without brand checks the read silently returns a no-op, so the captured "focus" does nothing (the Zag failure).

**The fix.** ~15 lines: the getter hands back the currently installed focus method for prototype reads and for any receiver that fails the brand check. Per-element behavior — recursion protection, the detached-iframe no-op — unchanged. Tests reproduce the full capture-and-wrap pattern end-to-end, red-green verified.

**The outcome.** [PR #35528](https://github.com/storybookjs/storybook/pull/35528) in review, linked from both issues.

**The takeaway.** A bug report's theory is a starting point, not a diagnosis. The stack trace doesn't lie — read it before you read the speculation.
