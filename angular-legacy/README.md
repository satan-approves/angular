# Angular Legacy Architecture (v12–16)

> NgModules · RxJS / BehaviorSubject · Zone.js · Class-based interceptors & guards

---

## Key Decisions

| Concern | Choice | Why |
|---|---|---|
| Components | NgModule-based | Standard for v12–16 codebases |
| State | BehaviorSubject in services | No extra deps, reactive via `async` pipe |
| Change detection | Zone.js (default) | Stable, well-understood |
| HTTP | Class-based interceptors | Standard pattern pre-v15 |
| Guards | Class-based (`CanActivate`) | Standard pattern pre-v15 |
| Forms | Reactive Forms | Type-safe, testable validators |
| Testing | Jest + TestBed | Angular's built-in test harness |
| UI | Angular Material | Accessible, themeable |

---

## Folder Structure

```
src/
  app/
    core/                        # Singleton module — imported once in AppModule
      core.module.ts             # Guards against re-import
      auth/
        auth.service.ts          # BehaviorSubject<User>, login/logout
        auth.guard.ts            # Class-based CanActivate
        auth.model.ts
      http/
        auth-token.interceptor.ts
        http-error.interceptor.ts

    shared/                      # SharedModule — imported in every feature module
      shared.module.ts           # Declares + exports all shared components
      components/
        button/
        modal/
        input/
      pipes/
        format-date.pipe.ts
        format-currency.pipe.ts
      directives/
        click-outside.directive.ts

    pages/                       # Feature modules — lazy-loaded
      payments/
        payments.module.ts         # Declares page + child components
        payments-routing.module.ts
        components/
          payment-form.component.ts
          payment-card.component.ts
        repository/
          payments.repository.ts   # HTTP only
        services/
          payments.service.ts      # Business logic
        state/
          payments.state.ts        # BehaviorSubject-based state
        validators/
          payment.validators.ts
        payments-page.component.ts # Smart/container component
      dashboard/
      profile/

    app.module.ts                # Root module
    app-routing.module.ts        # Root routes
    app.component.ts
  main.ts
```

---

## Data Flow

```
Template
  └── subscribes via async pipe (payments$ | async)
  └── calls methods on PaymentsService

PaymentsService
  └── calls PaymentsRepository (HTTP Observable)
  └── updates PaymentsState (BehaviorSubject)

PaymentsState
  └── holds BehaviorSubjects: _payments$, _isLoading$, _error$
  └── exposes read-only Observables: payments$, isLoading$, error$
```

---

## State Example

```ts
// state — BehaviorSubject, exposed as Observable
@Injectable({ providedIn: 'root' })
export class PaymentsState {
  private _payments$ = new BehaviorSubject<Payment[]>([]);
  payments$ = this._payments$.asObservable(); // read-only outside

  setPayments(payments: Payment[]) { this._payments$.next(payments); }
}

// template — async pipe handles subscribe/unsubscribe
<div *ngFor="let p of payments$ | async">{{ p.amount }}</div>
```

---

## Module Pattern

- **CoreModule** — imported once in `AppModule`. Guards against re-import.
- **SharedModule** — imported in every feature module that needs shared components.
- **Feature modules** — lazy-loaded, self-contained. Each declares its own components.

---

## Scalability Notes

- **Add a page**: create a new feature module + routing module, add a lazy route in `app-routing.module.ts`
- **Shared component**: add to `SharedModule` declarations + exports
- **Cross-page state**: promote to a `core/` service
- **Complex global state**: introduce NgRx per feature
