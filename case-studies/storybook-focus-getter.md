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

**The third issue — and empirical proof.** A week later a *third* issue appeared: Ark UI / Zag.js overlays with `defaultOpen` parking off-screen in Storybook 10.5, in the plain preview (no test addon). The reporter had done careful work and *explicitly ruled out the focus patch* — polling the property descriptor 81 times, never seeing an accessor. It looked like a separate regression.

I rebuilt the scenario from scratch: a `storybook build` with Storybook 10.5.3 + Ark UI, inspected with Playwright. The key turned out to be *which component*. A `Popover` (doesn't move focus) positioned fine; a `Menu` (manages focus) parked at `-100vh`. The captured stack:

```
TypeError: Illegal invocation
    at HTMLElement.get [as focus]   // the same 10.5 getter
    at trackFocusVisible            // @zag-js/focus-visible
```

`@zag-js/focus-visible` reads `HTMLElement.prototype.focus` to capture-and-wrap it — the exact prototype read from the original diagnosis. The thrown error aborts Zag's setup *before floating-ui runs*, so the positioner never leaves its initial `-100vh`. The reporter's ruling-out was reasonable: they'd tested a component that doesn't call `trackFocusVisible`.

Then I proved the fix rather than asserting it: patched the built bundle with #35528's guard and re-ran the Menu story. Before: `matrix(1,0,0,1,0,-720)`, `pointer-events: none`, thrown error. After: `matrix(1,0,0,1,16,45)`, `pointer-events: auto`, no error. One fix, three issues (#35503, #35502, #35546), with a reproducible before/after.

**The takeaway.** A bug report's theory is a starting point, not a diagnosis — the stack trace doesn't lie. And when you claim a fix resolves an issue nobody connected to it, *prove it end-to-end* rather than reasoning by analogy: a patched bundle and a before/after measurement turn "should fix it" into "does fix it."
