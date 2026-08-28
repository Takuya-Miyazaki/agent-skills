# React View Transitions — Agent Summary

Read [SKILL.md](SKILL.md) first. It links current React and Next.js documentation and routes detailed work to focused references.

Quick rules:

- Let React own `document.startViewTransition`; use React Transitions, Suspense, deferred updates, or Next.js navigation as the trigger.
- Use globally unique names only for real identity across snapshots.
- Keep route enter/exit VTs in pages unless the layout itself should animate.
- Test prefetched and suspended routes separately.
- For reveal-only crossfades, keep `Crossfade` wrapper-free and add a DOM host outside `Suspense` only at affected navigable call sites.
- Keep optimistic feedback separate from the committed element that owns a shared name.
- Isolate persistent chrome and portaled UI; use an always-mounted named owner for third-party portals.
- Disable the root animation when unnamed app chrome must stay live.
- Prefer opacity-only crossfades and always support reduced motion.

Detailed references:

- [Implementation workflow](references/implementation.md)
- [Pattern catalog](references/patterns.md)
- [Next.js integration](references/nextjs.md)
- [CSS recipes](references/css-recipes.md)
