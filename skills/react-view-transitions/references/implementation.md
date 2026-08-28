# Implementation Workflow

Use this workflow for a whole-app rollout or a broad review. For API behavior, read the official documentation linked from [`SKILL.md`](../SKILL.md).

## 1. Audit Before Editing

Search for existing transition code and navigation triggers:

```bash
rg -n "ViewTransition|viewTransitionName|transitionTypes|addTransitionType|startTransition|Suspense|useDeferredValue" app components features
```

Record each candidate in a small table:

| Surface | Trigger | Lifecycle | Intended meaning |
|---|---|---|---|
| Route content | navigation | replaced | hierarchy or reveal |
| Card/detail media | navigation | paired | same object |
| Filtered list | deferred update | persists | items moved |
| Modal or menu | transition | mounts/unmounts | appeared/disappeared |
| Player/nav/toast | unrelated | persists | should stay still/live |

Do not infer lifecycle from component names. Inspect where the boundary mounts relative to layouts, Suspense, conditionals, portals, and keyed children.

## 2. Establish One Transition

Implement the smallest semantic layer first:

- shared identity for the same object across views;
- enter/exit for a component appearing or disappearing;
- update for mounted content or list movement;
- typed navigation for hierarchy;
- reveal for suspended content.

Verify that layer before adding another. Nested boundaries can claim the mutation that an outer boundary expected to animate.

## 3. Handle Persistent UI

Audit fixed or portaled surfaces before enabling broad route motion:

- navigation bars and sidebars;
- audio/video players;
- theme or demo toolbars;
- dialogs, popovers, menus, and tooltips;
- toast providers.

Isolate only elements that visually misbehave. If a third-party portal mounts its own DOM, put its provider/output inside an always-mounted named owner rather than relying on the portal's transient node. See [Persistent UI and portals](patterns.md#persistent-ui-and-portals).

## 4. Add Suspense Deliberately

For every animated boundary, test two distinct paths:

1. Content is ready and no fallback appears.
2. The fallback commits and content reveals later.

If an enter-only content VT flashes on the first path, use the call-site host pattern in [Warm navigation versus cold reveal](patterns.md#warm-navigation-versus-cold-reveal). Do not add the host globally; existing DOM structure may already provide it.

Check fallback and content for duplicated headings, controls, or media. Move stable UI outside the boundary when possible.

## 5. Add CSS by Pattern

Copy only the relevant section from [CSS recipes](css-recipes.md). Avoid a global animation rule that accidentally animates every named element.

At minimum, include reduced motion. Add the live-root rule only when persistent unnamed UI freezes or blocks interaction. Add portal isolation only for surfaces that need it.

## 6. Verify the Matrix

For each route or interaction, verify:

- warm/prefetched destination;
- cold/suspended destination;
- forward navigation and browser history;
- same-route parameter or query changes;
- optimistic and committed states;
- revalidation after the screen is already visible;
- open overlays and pointer input during the animation;
- reduced motion.

Use a recording for brief flicker and geometry bugs. Screenshots alone often miss double snapshots and momentary opacity dips.

## Review Checklist

- [ ] The animation communicates a specific relationship.
- [ ] Every name is unique among simultaneously mounted elements.
- [ ] Shared pairs exist in the same transition.
- [ ] `default="none"` has not disabled a needed trigger.
- [ ] Type maps have a `default` path.
- [ ] Suspense reveals do not depend on navigation types.
- [ ] Persistent and portaled UI stays stable and interactive.
- [ ] Warm and cold paths both behave intentionally.
- [ ] Unrelated updates do not animate.
- [ ] Reduced motion is present.
