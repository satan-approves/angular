# Angular Architecture

> Two flavors of the same app — pick the one matching your stack.

This repo is a **structure sketch**, not a runnable project. It shows how I'd lay out an Angular app, where layers live, and how data flows. No build, no tests, no `ng new` output — just folders and two READMEs.

---

## Why two flavors?

Angular changed significantly around v14–v17. Rather than guess which era your codebase is on, I sketched both:

| Folder | Angular era | Stack |
|---|---|---|
| [`angular-legacy/`](./angular-legacy/README.md) | v12–v16 | NgModules · RxJS / BehaviorSubject · Zone.js · class-based guards & interceptors |
| [`angular-modern/`](./angular-modern/README.md) | v17+ | Standalone components · Signals · Zoneless-ready · functional guards & interceptors |

Same app shape, same layering (repository → service → store → component), different mechanics. I'd match whatever the team is currently on; if we're on v14+ I'd lean modern.

---

## What's the same in both

- `core/` — singleton services (auth, HTTP interceptors, layout)
- `components/` (or `shared/`) — reusable dumb components
- `pages/` — one folder per feature, self-contained, lazy-loaded
- Inside a page: `repository/` (HTTP only), `services/` (business logic), `store/` or `state/` (local state), `validators/`, a smart page component, a routes file, a barrel `index.ts`

**Business logic lives in services, not components.** Components display and react.

---

## How to read this

1. Open the README in each flavor — decisions and data flow are explained there.
2. Click through the folder tree — the names tell the story.
3. Files are empty on purpose. This is about *where things go*, not *how they're written*.
