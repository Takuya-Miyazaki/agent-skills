# Production Patterns

This catalog contains patterns found while building and debugging real React and Next.js applications. It supplements rather than replaces the official React documentation linked from [`SKILL.md`](../SKILL.md).

## Warm Navigation Versus Cold Reveal

An enter-only VT inside Suspense can behave differently depending on whether navigation data was prefetched.

Keep the reusable component wrapper-free:

```tsx
export function Crossfade({ children }: { children: React.ReactNode }) {
  return (
    <ViewTransition enter="auto" default="none">
      {children}
    </ViewTransition>
  );
}
```

At a navigable call site that otherwise exposes `Suspense` at its root, add a persistent DOM host outside the boundary:

```tsx
<div>
  <Suspense fallback={<Skeleton />}>
    <Crossfade>
      <Content />
    </Crossfade>
  </Suspense>
</div>
```

Why it works:

- On a warm navigation, React inserts the host and suppresses the nested enter.
- On a cold path, the host mounts with the fallback and persists; the nested VT enters only when content resolves.

Do not put the host inside `Crossfade`; that suppresses the reveal too. Audit call sites before editing because many already have a direct DOM host. A broad ancestor that persists across navigation is not a substitute.

Use a fallback/content shared pair only when an actual geometry morph is desired. It is not required for a reveal fade.

## Appended Async Pages

The initial page is already part of the screen. Only later pages represent new content arriving after interaction:

```tsx
{pages.map((page, index) => (
  <Suspense key={index} fallback={<RowsSkeleton />}>
    {index === 0 ? <Page page={page} /> : <Crossfade><Page page={page} /></Crossfade>}
  </Suspense>
))}
```

This avoids replaying an arrival animation for the first page after state resets, navigation, or hydration.

## Optimistic State with a Committed Shared Indicator

Optimistic state should make the control feel immediate, but a named shared element needs a single stable source and destination. Use optimistic state for labels, colors, and pending treatment; render the named indicator from committed route state:

```tsx
const [optimisticTab, setOptimisticTab] = useOptimistic(activeTab);

{tabs.map(tab => {
  const isActive = optimisticTab === tab.value;
  const isCommitted = activeTab === tab.value;

  return (
    <Link
      key={tab.value}
      href={tab.href}
      aria-current={isActive ? 'page' : undefined}
      onNavigate={() => startTransition(() => setOptimisticTab(tab.value))}
    >
      {tab.label}
      {isCommitted ? (
        <ViewTransition name="tab-indicator" share="tab-underline">
          <span aria-hidden />
        </ViewTransition>
      ) : null}
    </Link>
  );
})}
```

If the indicator follows optimistic state, the old named element can disappear before the route commits, leaving no reliable shared pair.

## Layout Displacement

When content changes height, the section below it moves. Wrapping only the changing content does not animate the displaced section; wrap the section that moves:

```tsx
<ChangingList />
<ViewTransition>
  <RelatedSection />
</ViewTransition>
```

Keep the moving boundary as a direct sibling where possible. Leave `update` enabled; `default="none"` disables it unless explicitly restored.

## Keyed Collections

Wrap each stable item rather than the entire collection when insertions, removals, or reordering should retain identity:

```tsx
{items.map(item => (
  <ViewTransition key={item.id}>
    <Row item={item} />
  </ViewTransition>
))}
```

React keys provide reconciliation identity. Add explicit VT names only when the item must pair with another representation elsewhere.

## Persistent UI and Portals

Fixed chrome and portaled overlays often appear in both snapshots and can double, fade, or freeze. Give affected surfaces stable names and disable their snapshot animation in CSS.

For a third-party portal, name an always-mounted owner:

```tsx
<div style={{ viewTransitionName: 'toaster' }} className="fixed inset-0 pointer-events-none">
  <ThirdPartyToaster />
</div>
```

This is more reliable than naming a transient node created inside the library. Use different names for independent simultaneous surfaces such as a modal and its backdrop.

Backdrop blur is a special case: the old and new translucent snapshots can stack and look darker. Hide the old backdrop snapshot rather than fading two blurred layers together. See [Persistent element isolation](css-recipes.md#persistent-element-isolation).

## Keep the Root Live

The default root crossfade snapshots the whole document. In app shells with persistent unnamed chrome, that can freeze hover states or briefly intercept interaction. Disable root animation and animate only named groups:

```css
::view-transition-old(root) {
  display: none;
}

::view-transition-new(root) {
  animation: none;
}
```

Also set `pointer-events: none` on the transition overlay when live controls must remain clickable. Use this as an app-shell policy, not an automatic default for every site.

## Duplicate Content Across Suspense

If fallback and content both render the same heading, toolbar, or media, the two snapshots can fade against each other and produce a visible dip.

Preferred order:

1. Render stable content outside Suspense.
2. If it genuinely changes identity, give the two versions a deliberate shared transition.
3. Otherwise keep that element out of the animated group.

## Same-Route Content Direction

For calendar-like views where the route shell persists and only a parameter changes, use a stable name plus a keyed child so React sees replacement, then choose `share` from the navigation type. See [Same-route dynamic segments](nextjs.md#same-route-dynamic-segments).

## Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| No animation | Update did not run in a React Transition/Suspense/deferred update | Fix the trigger; do not call the browser API directly |
| Shared morph does not run | Pair was not present in the same transition, name differs, or `share` resolved to `none` | Inspect both snapshots, prefetch/cache if appropriate, and verify the type path |
| Duplicate-name warning | Two mounted instances own the same global name | Make the name conditional or include stable identity |
| Prefetched content flashes | Content-side enter VT became the topmost inserted subtree | Add a call-site host outside Suspense |
| Suspense content never fades | A host was added inside the reusable Crossfade | Move the host outside Suspense at the affected usage |
| Heading dips on reveal | Fallback and content both render it | Move it outside the boundary or intentionally share it |
| Section teleports after list changes | Only the list was wrapped | Wrap the displaced sibling and keep update enabled |
| Modal/backdrop becomes darker | Two translucent snapshots overlap | Hide the old snapshot for that group |
| Toast or menu flickers | The library's portal node mounts transiently | Name an always-mounted owner around the portal output |
| UI freezes during animation | Root snapshot covers persistent chrome | Use the live-root and pointer-event rules |
| `viewTransitionName: 'none'` changes nothing | `none` is the absence of a name | Assign a real unique name or remove the property |
| Browser Back skips direction | History traversal has no transition type | Provide a calm default; keep shared morphs untyped when they should still run |
