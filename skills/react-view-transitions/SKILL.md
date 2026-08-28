---
name: vercel-react-view-transitions
description: Guide for implementing and debugging React View Transition animations, including shared elements, Suspense reveals, list and layout updates, directional navigation, and Next.js App Router integration. Use when a task mentions ViewTransition, startViewTransition, addTransitionType, transitionTypes, crossfades, route animations, or shared-element morphs.
license: MIT
metadata:
  author: vercel
  version: "1.0.0"
---

# React View Transitions

Use React's View Transition integration to communicate continuity, hierarchy, or content arrival. Do not add motion without a clear meaning.

## Read Current Documentation First

The APIs are evolving. Use the official references instead of reproducing their API documentation from memory:

- [React `<ViewTransition>`](https://react.dev/reference/react/ViewTransition)
- [React `addTransitionType`](https://react.dev/reference/react/addTransitionType)
- [Next.js View Transitions guide](https://nextjs.org/docs/app/guides/view-transitions)
- [Next.js `<Link>` `transitionTypes`](https://nextjs.org/docs/app/api-reference/components/link#transitiontypes)
- [Next.js `useRouter`](https://nextjs.org/docs/app/api-reference/functions/use-router)

For Next.js, also read the matching guide under `node_modules/next/dist/docs/`. The installed version is authoritative for configuration and signatures.

## Workflow

1. Audit the update trigger, participating VTs, mount/persist/unmount behavior, Suspense boundaries, and persistent chrome.
2. Implement the smallest semantic pattern first.
3. Add only the CSS recipes that pattern needs.
4. Verify warm and cold navigation, history traversal, fallback reveals, unrelated updates, and reduced motion.

For a whole-app rollout, follow [references/implementation.md](references/implementation.md). For focused work, load only the relevant reference:

| Need | Reference |
|---|---|
| Production patterns | [references/patterns.md](references/patterns.md) |
| Unexpected or broken animation behavior | [references/troubleshooting.md](references/troubleshooting.md) |
| Next.js routing and prefetch behavior | [references/nextjs.md](references/nextjs.md) |
| Reusable CSS | [references/css-recipes.md](references/css-recipes.md) |

## Choose by Meaning

| Intent | Pattern |
|---|---|
| Same object across views | Matching `name` and `share` |
| Data arrived | Suspense reveal |
| Same items moved | Keyed item VTs with `update` enabled |
| Component appeared/disappeared | `enter` / `exit` |
| Hierarchical navigation | Type-keyed forward/back animation |
| Same place, different content | Update or shared crossfade |

Shared identity and list identity are separate. A list item containing a shared image can need an outer keyed VT for reorder and an inner named VT for the cross-route morph.

## Guardrails

- Let React call `document.startViewTransition`; do not call it directly in React code.
- State-driven VTs need a React Transition, Suspense, or deferred update. Ordinary `setState` does not activate them.
- Treat `name` as global identity. It must be unique among simultaneously mounted elements.
- `default="none"` disables unspecified triggers, including `update` and `share`; opt required triggers back in.
- Type maps need a `default`. Suspense reveals happen in a later untyped transition, so reveal VTs normally use string props.
- A raw `viewTransitionName` isolates an element; it does not start a transition. `viewTransitionName: "none"` is no isolation.
- Keep route enter/exit boundaries in pages unless a persistent layout is intentionally the animated subject.
- Prefer opacity-only crossfades. Reserve blur for a deliberate morph.
- Always include reduced-motion CSS.

## Patterns Learned from Production Apps

The detailed implementations live in the references; these are the cases most often missed:

- **Optimistic controls:** update labels and pending styles optimistically, but keep named shared indicators tied to committed route state.
- **Layout displacement:** wrap the sibling that moves, not only the content whose size changed.
- **Persistent and portaled UI:** isolate fixed chrome and floating surfaces with stable names. For third-party portals, name an always-mounted owner.
- **Fallback/content duplicates:** move repeated headings and controls outside Suspense, or give them deliberate shared identity, to avoid an opacity dip.
- **Live app shells:** disable the root snapshot animation when unnamed persistent chrome must stay live and interactive.

## Verification

Check the behavior users see:

- prefetched/warm navigation and cold/suspended navigation;
- forward links, programmatic navigation, and browser back/forward;
- every path where a shared pair should or should not form;
- optimistic and committed states;
- persistent chrome, open overlays, pointer interaction, and reduced motion;
- revalidation and unrelated transitions, which should remain quiet.

Use screenshots or recordings when timing, geometry, or flicker is the issue.

## Full Compiled Document

For the complete standalone guide with all references expanded: `AGENTS.md`.
