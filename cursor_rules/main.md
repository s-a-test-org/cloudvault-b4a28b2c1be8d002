## AI Entry Point

Use this file to quickly locate the right documentation for server and client code in this repository. If you are searching for a specific topic, follow the sections below to jump to the correct guide.


### What is Rhino?

Rhino is an opinionated Rails framework that provides resource modeling, properties-driven APIs, authentication/authorization with policies, and a set of conventions for building maintainable backends and consistent frontends. It integrates with Pundit-based policies and auto-generates schema and validations from model properties.

- Properties overview: [Properties](https://www.rhino-project.org/docs/guides/properties)
- Policies overview: [Policies](https://www.rhino-project.org/docs/concepts/auth/policies)


### Server documentation (backend)

Location: `./server/`

- `./server/models.md`: Rhino model conventions, ownership, references, properties, search, attachments, nested attributes, synthetic/generated attributes, validations, and examples.
- `./server/policies.md`: Role-based policies, naming, choosing base policies, required scopes, `auth_owner` usage, strong params, generators, and best practices.
- `./server/controllers.md`: Thin controllers, Rhino vs API controllers, authorization patterns, endpoint comments, routing examples, and a concise checklist.
- `./server/notifications.md`: Notification templates, how `notify_later` keys map to view paths, HTML/text template examples, and implementation checklist. For model hooks, see `./server/models.md`.
- `./server/data-model.md`: System-wide data model notes (add high-level ERD, glossary, and cross-cutting model relationships here).

Tip: If you are implementing a new resource end-to-end, start with `./server/models.md`, then `./server/policies.md`, and finally wire custom endpoints using `./server/controllers.md`.


### Client documentation (frontend/mobile)

Location: `./web/` or `./mobile/`

Use this area to document client-side conventions and components. Typical structure:

- Web (Rhino frontend): architecture, routing, UI components, and integration points with server resources.
- Mobile (Expo/React Native): navigation layout, screens, shared UI primitives, and API integration.

- `./web/component-override.md`: How to customize Rhino UI via `app/frontend/rhino.config.jsx` and per-model overrides, including global Displays/Fields, Index/Show/Create/Edit surfaces, Sidebar/Shell, and upstream component references. Use this when you need to override headers, forms, tables, filters, cells, or displays.
- `./web/rhino-api.md`: How to use Rhino’s frontend API hooks for queries and mutations (`useModelIndex`, `useModelShow`, `useModelCreate/Update/Destroy`), plus recipes and patterns for custom controller calls. Use this when fetching, mutating, or composing data flows in the UI.

### How to find what you need

- Need to expose a new model to the API? See `./server/models.md`.
- Need to authorize access or scope queries? See `./server/policies.md`.
- Need to add a custom endpoint or route? See `./server/controllers.md`.
- Need to send user notifications? See `./server/notifications.md`.
- Need client-side integration details? Check `./web/` or `./mobile/` and source folders above.

- Need to override UI components (inputs, tables, headers, displays)? See `./web/component-override.md`.
- Need to fetch or mutate data in the frontend (hooks, custom calls)? See `./web/rhino-api.md`.


