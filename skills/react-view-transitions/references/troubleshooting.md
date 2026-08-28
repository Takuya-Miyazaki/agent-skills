# Troubleshooting

**VT not activating:** Ensure `<ViewTransition>` comes before any DOM node. Ensure state update is inside `startTransition`.

**"Two ViewTransition components with the same name":** Names must be globally unique. Use IDs: `name={`hero-${item.id}`}`.

**Scrolling hangs while a transition animates:** the `::view-transition` overlay is `position: fixed` and its snapshots don't scroll — a browser limitation, not fixable in React (skipping snaps to the end). Keep reveal durations short; for scroll-driven UI use gesture transitions (experimental `useSwipeTransition`, if available).

**Open popover flickers when a background transition settles:** it's captured in `root`. Give it a real `view-transition-name` + isolation (not `none`) — see [Isolate Elements from Parent Animations](patterns.md#isolate-elements-from-parent-animations).

**Popover closes or goes dead when clicked mid-transition:** named participants are skipped by hit-testing while a transition runs; clicks land on what's beneath and read as outside-clicks. Portal the popover (see [Isolate Elements from Parent Animations](patterns.md#isolate-elements-from-parent-animations)); brief dead clicks during the transition remain — that's the price of the name.

**Shared morph silently not firing:** `share` resolved to `none`. Either the VT has `default="none"` with no explicit `share` prop, or `share` is type-keyed and the navigation never adds the type — the link needs `transitionTypes` (or `addTransitionType` in the transition).

**An enter animation runs even though Suspense never showed its fallback:** The content-side VT became the topmost entering subtree during navigation. At that call site, add a host DOM element immediately outside `Suspense`. The host suppresses the nested enter during warm navigation but remains mounted so the VT can enter on a later fallback-to-content reveal. Keep the host outside `Suspense`, not inside the reusable crossfade; do nothing when a direct host already exists.

**Section below a list teleports instead of gliding:** it's outside any activated boundary, its VT has `default="none"` (which disables `update`), or it isn't an immediate sibling of the changing content. See [Layout Displacement Morph](patterns.md#layout-displacement-morph).

**`router.back()` and browser back/forward skip the directional slide:** traversals carry no transition types, so type-keyed maps resolve to `default` — untyped shared-element morphs still apply. Use `router.push()` for typed animations.

**`flushSync` skips animations:** Use `startTransition` instead.

**Only updates animate (no enter/exit):** Without `<Suspense>`, React treats swaps as updates. Conditionally render the VT itself, or wrap in `<Suspense>`.

**Layout VT prevents page VTs from animating:** nested VTs skip their own enter/exit only when they mount/unmount *as one unit* with a parent VT. If page enter/exit is dead under a persistent layout VT, the usual culprit is a DOM node above the page VT. Keep VTs in pages, not layouts.

**List reorder not animating with `useOptimistic`:** Optimistic values resolve before snapshot. Use committed state for list order.

**TS error "Property 'default' is missing":** Type-keyed objects require a `default` key.

**Hash fragments cause scroll jumps:** Navigate without hash; scroll programmatically after navigation.

**Backdrop-blur flickers:** Use the [Backdrop-Blur Workaround](css-recipes.md#backdrop-blur-workaround).

**`border-radius` lost during transitions:** Apply `border-radius` directly to the captured element.

**Skeleton controls slide away:** Give matching controls the same `viewTransitionName`.

**Batching:** Multiple updates during animation are batched. A→B→C→D becomes B→D.
