# Angular Modern Architecture (v17+)

> Standalone components · Signals · Zoneless · Functional interceptors & guards

---

## Key Decisions

| Concern | Choice | Why |
|---|---|---|
| Components | Standalone (no NgModules) | Less boilerplate, self-contained |
| State | Signals in services | Simple, reactive, no extra deps |
| Change detection | Zoneless-ready | Better perf, cleaner debugging |
| HTTP | Functional interceptors | Composable, no class overhead |
| Guards | Functional (`CanActivateFn`) | Simpler than class-based guards |
| Forms | Reactive Forms | Type-safe, testable validators |
| Testing | Jest + Angular Testing Library | Familiar from React ecosystem |
| UI | Angular Material | Accessible, themeable |

---

## Folder Structure

```
src/
  app/
    core/                        # Singleton services — loaded once globally
      auth/
        auth.model.ts            # User interface, AuthState
        auth.service.ts          # currentUser signal, login/logout
        auth.guard.ts            # Functional guard
      http/
        auth-token.interceptor.ts  # Attaches Bearer token to every request
        http-error.interceptor.ts  # Global error handling (401 redirect, toasts)
      layout/
        header.component.ts
        sidebar.component.ts

    components/                  # Shared dumb components — used across pages
      button/
      modal/
      input/

    pages/                       # Feature pages — self-contained units
      payments/
        components/              # Components used only on this page
          payment-form.component.ts
          payment-card.component.ts
        repository/
          payments.repository.ts # HTTP only — maps API ↔ model
        services/
          payments.service.ts    # Business logic — calls repo, updates store
        store/
          payments.store.ts      # Signals-based state
        validators/
          payment.validators.ts  # Pure validator functions
        payments.page.ts         # Smart/container component (entry point)
        payments.routes.ts       # Lazy-loaded route config
        index.ts                 # Public API — only export what other pages need
      dashboard/
      profile/

    app.component.ts
    app.config.ts                # provideRouter, provideHttpClient, etc.
    app.routes.ts                # Root routes with lazy loading
  main.ts
```

---

## Data Flow

```
Template
  └── reads store signals directly (payments.store.ts)
  └── calls methods on PaymentsService

PaymentsService
  └── calls PaymentsRepository (HTTP)
  └── updates PaymentsStore (signals)

PaymentsStore
  └── holds signals: payments, isLoading, error
  └── exposes computed: totalAmount, etc.
```

---

## State Example

```ts
// store — signals only, no subscriptions
@Injectable({ providedIn: 'root' })
export class PaymentsStore {
  payments = signal<Payment[]>([]);
  isLoading = signal(false);
  totalAmount = computed(() => this.payments().reduce((s, p) => s + p.amount, 0));
}

// template — reads signal directly
@for (payment of store.payments(); track payment.id) { ... }
```

---

## Scalability Notes

- **Add a page**: create a new folder under `pages/`, add a route in `app.routes.ts`
- **Shared component**: goes in `components/` with its own subfolder
- **Cross-page state**: promote to a `core/` service
- **Complex global state**: introduce NgRx Signal Store per feature
