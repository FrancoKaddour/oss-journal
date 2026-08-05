# Case study: one PR, two bugs — Floating Panel stacking in Zag.js

**The problem.** Zag.js — the framework-agnostic state machine library behind Chakra UI and Ark UI — shipped a Floating Panel whose stacking model broke down with more than one panel open. Closing the topmost panel left it marked `data-topmost` while the remaining panel stayed `data-behind` ([#3243](https://github.com/chakra-ui/zag/issues/3243)), and focusing a panel updated its state but never actually raised it above its siblings ([#3242](https://github.com/chakra-ui/zag/issues/3242)).

**The investigation.** Two symptoms, one subsystem. Panels joined a shared `panelStack` on open but only left it on machine *destroy* — closing wasn't leaving the stack, so nobody else got promoted. And the stack index (`--z-index`) was written only to the panel's content element, but every positioner is a sibling stacking context: by CSS's own rules, no z-index on a child can reorder its parent's siblings. The state was right; it just couldn't reach the layer that mattered.

**The fix.** Remove the panel from the stack when it closes (re-adding on reopen came free from the existing open-state action), and apply the stack index to the positioner, consumed via `zIndex: var(--z-index)`. Shipped with a new two-panel example page and two e2e regression tests asserting `data-topmost` and the computed positioner z-index — verified red-green locally, rebuilding the package in each direction.

**The outcome.** Segun Adebayo (creator of Chakra UI and Zag.js) reviewed by pushing a refactor commit *directly onto my branch* — keeping both mechanisms of the fix, making the stack index a reactive machine binding — and merged it the next morning ([#3246](https://github.com/chakra-ui/zag/pull/3246)). Both issues closed by one PR. My second merge in Zag.js in eight days.

**What I'd tell you in an interview.** The insight wasn't in the state machine — the reporter had already traced that. It was recognizing that the second bug was a CSS stacking-context problem wearing a state-management costume, and shipping both fixes as one coherent change with tests that prove the visual behavior, not just the state.
