# Next.js Integration

Use the installed Next.js documentation as the primary source. Current App Router releases need no experimental configuration.

Official references:

- [View Transitions guide](https://nextjs.org/docs/app/guides/view-transitions) — shared elements, Suspense reveals, directional routes, persistent UI, interaction, and same-route transitions
- [`<Link>` `transitionTypes`](https://nextjs.org/docs/app/api-reference/components/link#transitiontypes) — typed declarative navigation
- [`useRouter`](https://nextjs.org/docs/app/api-reference/functions/use-router) — typed `push` and `replace`, prefetching, and history navigation

Do not duplicate those examples in an implementation plan. Use the notes below for integration decisions the reference guide does not fully settle for a real app.

## Match the Installed Version

Read `node_modules/next/dist/docs/01-app/02-guides/view-transitions.md` and the relevant local API file before editing. The installed version is authoritative for flags and signatures.

## Boundary Placement

Keep route enter/exit VTs in pages unless the persistent layout itself is the animated subject. A layout survives navigation, and a boundary around `{children}` can become the topmost participant that suppresses nested page behavior.

Server Components can render `<ViewTransition>` and Suspense directly. Add a Client Component only for interaction, optimistic state, or other client-only behavior.

## Warm and Cold Navigation Are Different Paths

A prefetched destination can commit with the navigation and form a shared pair. A cold destination can commit a fallback first, so the pair does not exist and the later reveal is a separate untyped transition.

Test both paths. If reveal-only content flashes only when warm, use [Warm navigation versus cold reveal](patterns.md#warm-navigation-versus-cold-reveal). Do not add a client fetch just to make an animation pair; use the app's normal cache and prefetch architecture.

## Compose Route Direction and Content Arrival

Directional route motion and Suspense arrival are separate layers:

- route pages use a type map with a deliberate `default`;
- revealed content uses a string `enter` value because the later Suspense transition has no navigation type.

Browser back/forward carries no link transition type, so it follows the default path. Keep a shared morph untyped if it should still work during history traversal.

## Routing-Driven Tabs

When tab state lives in the URL, use links for navigation and prefetching. Optimistic state can update labels, color, and pending UI immediately, while the named shared indicator remains attached to committed route state. See [Optimistic state with a committed shared indicator](patterns.md#optimistic-state-with-a-committed-shared-indicator).

Avoid an extra panel-wide update transition when keyed or shared children already animate the content change; nested boundaries may claim the mutation.

## Same-Route Dynamic Segments

Follow the official guide's stable-name plus changing-key pattern. If the animation direction varies, map `share` by transition type and provide `default: "none"`. This is useful for calendar periods and other persistent shells where only a segment or query changes.

## `loading.tsx` and Explicit Suspense

`loading.tsx` provides a route-level Suspense boundary. Use explicit boundaries for regions that should stream or animate independently. Do not animate both layers automatically; identify which boundary owns the intended reveal.

## Query-Driven Filtering

Keep router hooks at component scope. Use `router.replace` for canonical URL state, and group optimistic control state with that navigation when the control should respond before the Server Component payload commits. Keep shared names tied to committed route state.
