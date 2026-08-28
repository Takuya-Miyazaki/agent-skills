---
name: vercel-react-view-transitions
description: Implement and debug React View Transition animations, including shared elements, Suspense reveals, list and layout updates, directional navigation, and Next.js App Router integration. Use when a task mentions ViewTransition, startViewTransition, addTransitionType, transitionTypes, crossfades, route animations, or shared-element morphs.
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# React View Transitions

Use React's View Transition integration to communicate continuity, hierarchy, or content arrival. Keep motion semantic and narrowly scoped.

## Start with the Source Documentation

Do not reproduce the API reference from memory. Check the current documentation for the APIs the task uses:

- [React `<ViewTransition>`](https://react.dev/reference/react/ViewTransition)
- [React `addTransitionType`](https://react.dev/reference/react/addTransitionType)
- [Next.js View Transitions guide](https://nextjs.org/docs/app/guides/view-transitions)
- [Next.js `<Link>` `transitionTypes`](https://nextjs.org/docs/app/api-reference/components/link#transitiontypes)

For a Next.js repository, also read the matching local guide in `node_modules/next/dist/docs/`. The installed version is authoritative when it differs from the website or prior knowledge.

## Workflow

1. Identify the update that should animate: a React Transition, Suspense reveal, deferred update, or Next.js navigation.
2. Inventory the participating elements, their mount/persist/unmount behavior, and any persistent chrome or portals.
3. Choose one semantic pattern; do not layer animations until the basic path is correct.
4. Add only the CSS recipe that pattern needs.
5. Verify warm and cold navigation, browser history, fallback reveals, unrelated updates, and reduced motion.

Load only the reference needed for the task:

| Need | Reference |
|---|---|
| Audit or whole-app rollout | [Implementation workflow](references/implementation.md) |
| Production patterns and debugging | [Pattern catalog](references/patterns.md) |
| Next.js routing and prefetch behavior | [Next.js integration](references/nextjs.md) |
| App-specific CSS refinements | [Supplemental CSS recipes](references/css-recipes.md) |

## Patterns Distilled from Production Apps

These are the non-obvious cases worth checking after reading the official docs:

- **Reveal-only Suspense fade:** keep the reusable `Crossfade` wrapper-free and add a persistent DOM host immediately outside `Suspense` only at navigable call sites that need it. This suppresses an enter on warm navigation while preserving a later cold reveal.
- **Appended async pages:** render the initial page normally; animate only pages appended after interaction.
- **Optimistic navigation:** update pending labels or colors immediately, but keep a named shared indicator attached to committed route state so there is only one stable shared pair.
- **Layout displacement:** wrap the moving sibling itself. A boundary around only the content whose size changed cannot animate the displaced section.
- **Persistent and portaled UI:** isolate mobile bars, players, toolbars, menus, modals, and toasts with stable names. Put a third-party portal inside an always-mounted named owner when its internal lifecycle is unreliable.
- **Fallback/content duplicates:** move repeated headings and controls outside Suspense, or give them an intentional shared identity, to prevent an opacity dip.
- **Interactive root:** disable the root snapshot animation when unnamed persistent chrome must remain live during a transition.

The implementations and failure modes are in [references/patterns.md](references/patterns.md).

## Guardrails

- Let React call `document.startViewTransition`; do not call it directly in React code.
- Trigger state-driven animations from a Transition, Suspense, or deferred update as documented by React.
- Treat `name` as global identity. It must be unique among simultaneously mounted elements.
- `default="none"` disables unspecified triggers, including `update` and `share`; opt required triggers back in.
- Type maps need a `default`. Suspense resolution is a later untyped transition, so reveal VTs normally use string props.
- A raw `viewTransitionName` isolates an element; it does not start a transition.
- `viewTransitionName: "none"` does not isolate anything.
- Prefer opacity-only crossfades. Use blur only when it is a deliberate part of a morph.
- Keep route-level enter/exit boundaries in pages unless a persistent layout is intentionally the animated subject.
- Next.js App Router View Transitions require no experimental config in current releases; verify against the installed docs.

## Verification

Test behavior, not only compilation:

- prefetched/warm navigation and cold/suspended navigation;
- forward links, programmatic navigation, and browser back/forward;
- shared pairs present, absent, and duplicated;
- open overlays, hover and pointer interaction, and persistent chrome;
- revalidation and unrelated transitions, which should remain quiet;
- `prefers-reduced-motion`.

Use browser screenshots or recordings when timing, geometry, or flicker is the issue.
