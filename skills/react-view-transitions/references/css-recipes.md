# Supplemental CSS Recipes

Start with the [complete CSS from the official Next.js guide](https://github.com/vercel-labs/react-view-transitions-demo/blob/main/src/app/globals.css) when implementing its shared-element, Suspense, or directional examples. Copy only the classes the app uses.

The recipes below cover production-app refinements that are easy to miss.

## Crisp Crossfade

Keep a normal crossfade opacity-only. Do not put blur in a shared fade keyframe because every reveal will inherit it.

```css
@keyframes vt-fade {
  from { opacity: 0; }
}

::view-transition-old(.fade-out) {
  animation: 120ms ease-in both reverse vt-fade;
}

::view-transition-new(.fade-in) {
  animation: 180ms ease-out 120ms both vt-fade;
}
```

Remove the delay from the new snapshot for a simultaneous crossfade. If a specific morph needs softness, give it a separate class and keyframe.

## Live Root and Interaction

Use this in app shells when unnamed persistent UI must remain live instead of sitting behind a root snapshot:

```css
::view-transition {
  pointer-events: none;
}

::view-transition-old(root) {
  display: none;
}

::view-transition-new(root) {
  animation: none;
}
```

This is an app-shell policy, not an automatic default for every site.

## Persistent Element Isolation

Assign stable names in JSX, then disable snapshot animation for affected groups:

```css
::view-transition-group(mobile-nav),
::view-transition-group(player),
::view-transition-group(modal),
::view-transition-group(modal-backdrop),
::view-transition-group(toaster) {
  animation: none;
  z-index: 100;
}

::view-transition-old(mobile-nav),
::view-transition-new(mobile-nav),
::view-transition-old(player),
::view-transition-new(player),
::view-transition-old(modal),
::view-transition-new(modal),
::view-transition-new(modal-backdrop),
::view-transition-old(toaster),
::view-transition-new(toaster) {
  animation: none;
}
```

### Translucent Backdrops

Two backdrop-blur snapshots stack and darken. Keep the new snapshot and hide the old one:

```css
::view-transition-old(modal-backdrop) {
  display: none;
}
```

Use different names for the dialog and backdrop so their stacking and old-snapshot behavior can differ.

## Shared Tab Indicator

```css
::view-transition-group(.tab-underline) {
  animation-duration: 200ms;
  animation-timing-function: cubic-bezier(0.22, 1, 0.36, 1);
}

::view-transition-old(.tab-underline),
::view-transition-new(.tab-underline) {
  height: 100%;
}
```

The named element should track committed route state. Optimistic state can update labels and colors around it.

## Reduced Motion

```css
@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) {
    animation-duration: 0s !important;
    animation-delay: 0s !important;
  }
}
```
