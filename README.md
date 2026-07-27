# OSS Journal

Daily working log of my open source contributions — what I investigated, what I found, and where it landed. Real entries only: if there's no entry for a day, I wasn't working on OSS that day.

**Contributing across:** [Storybook](https://github.com/storybookjs/storybook) · [TanStack](https://github.com/TanStack/router) · [Vite](https://github.com/vitejs/vite) · [supabase-js](https://github.com/supabase/supabase-js) · [Ark UI / Zag.js](https://github.com/chakra-ui/zag)

## Highlights

**Merged & shipped**
- **supabase-js** — realtime `track()` opts fix, shipped in [v2.110.6](https://github.com/supabase/supabase-js/releases/tag/v2.110.6) (credited in release notes)
- **supabase-js** — functions abort-listener cleanup, shipped in [v2.110.8](https://github.com/supabase/supabase-js/releases/tag/v2.110.8) (credited in release notes)
- **Zag.js** — [#3233](https://github.com/chakra-ui/zag/pull/3233): fixed viewport-dependent marquee speed, **merged by the project creator** (Segun Adebayo), queued for the next release

**In review**
- **Storybook** — [#35528](https://github.com/storybookjs/storybook/pull/35528): root-caused a `TypeError` in the 10.5 focus instrumentation and proved (with a patched-bundle before/after) that it resolves **three** separate issues — #35503, #35502, #35546. See the [case study](./case-studies/storybook-focus-getter.md).
- **Storybook** — [#35527](https://github.com/storybookjs/storybook/pull/35527) (JSDoc parser) · [#35544](https://github.com/storybookjs/storybook/pull/35544) (dev-UI coverage race)
- **TanStack Router** — [#7817](https://github.com/TanStack/router/pull/7817) (lifecycle hooks feature)

**Triage & investigations** (reproductions + root-cause analysis)
- **TanStack Router** — [#7837](https://github.com/TanStack/router/issues/7837) (prerender query corruption, with minimal repro) · [#7829](https://github.com/TanStack/router/issues/7829) (`wrapInSuspense` parity)
- **Vite** — [#22968](https://github.com/vitejs/vite/issues/22968), [#23007](https://github.com/vitejs/vite/issues/23007) (bundled-dev behavior)
- **Zag.js** — [#3203](https://github.com/chakra-ui/zag/issues/3203) (dialog backdrop pointer-events)

Entries live under [`journal/`](./journal), one file per day. Deeper write-ups live under [`case-studies/`](./case-studies).
