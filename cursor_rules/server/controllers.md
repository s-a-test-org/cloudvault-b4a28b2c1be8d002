## Controllers: Authoring Guide for AI

This document describes how to implement controllers in a Rhino-powered Rails application. Controllers must be thin and simple; move business logic to models and helpers. Use clear endpoint comments at the start of each action like `# GET /api/example/items`.


### Controller types

- Rhino resource controllers (model-backed) live under `app/controllers/rhino` and are defined inside the `Rhino` module.
- Non-model API controllers live under `app/controllers/api` and are defined inside the `Api` module.

Keep consistent structure, explicit authorization, and clear parameter validation in all controllers.


### Rhino controllers (model-backed)

Model-backed endpoints should extend Rhino’s base Crud controller and use Pundit authorization with `policy_scope` and `authorize`. The routes typically live under a scoped path that includes `api/...` for consistency with frontend expectations.

```ruby
# frozen_string_literal: true

module Rhino
  class WorkPackagesController < CrudController
    # POST /api/work_packages/:id/initialize
    def initialize
      work_package = find_resource(policy_scope(klass))
      authorize work_package, :update?
      WorkPackageHelper.initialize_work(work_package, current_user)
      render status: :ok, json: work_package
    end

    # POST /api/work_packages/:id/set_status
    def set_status
      work_package = find_resource(policy_scope(klass))
      authorize work_package, :update?
      work_package.update(status: params[:status])
      render status: :ok, json: work_package
    end

    # POST /api/work_packages/:id/finalize
    def finalize
      work_package = find_resource(policy_scope(klass))
      authorize work_package, :update?
      WorkPackageHelper.finalize(work_package)
      render status: :ok, json: work_package
    end
  end
end
```

Routing examples for Rhino controllers:

```ruby
# frozen_string_literal: true

Rails.application.routes.draw do
  scope module: 'rhino' do
    post 'api/work_packages/:id/initialize', to: 'work_packages#initialize', as: 'work_packages_initialize', rhino_resource: 'WorkPackage'
    post 'api/work_packages/:id/set_status', to: 'work_packages#set_status', as: 'work_packages_set_status', rhino_resource: 'WorkPackage'
    post 'api/work_packages/:id/finalize', to: 'work_packages#finalize', as: 'work_packages_finalize', rhino_resource: 'WorkPackage'
  end
end
```

Notes:
- Use `find_resource(policy_scope(klass))` to locate the record within the authorized scope.
- Call `authorize record, :action?` for each custom action.
- Keep controller actions short; push domain logic to helpers or models.


### API controllers (non-model endpoints)

Non-model endpoints belong under `app/controllers/api` within the `Api` module. These do not use Rhino’s internal Crud controllers, so you must explicitly enforce access inside the controller (e.g., with a `before_action`).

```ruby
# frozen_string_literal: true

module Api
  class MetricsDashboardController < ApplicationController
    before_action :authorize_access

    # GET /api/metrics_dashboard
    def index
      # call some helper... do something

      render status: :ok, json: MetricsHelper.build_dashboard(
        organization_id: organization_id,
        from: from,
        to: to
      )
    end

    private
      def authorize_access
        # Ensure the signed-in user is authorized to use this API endpoint
        # Example: limit to members of a specific organization or feature flag
        return if current_user&.organizations&.exists?(id: params[:organization_id])

        render status: :forbidden, json: { error: 'Forbidden' }
        nil
      end
  end
end
```

Routing examples for API controllers:

```ruby
# frozen_string_literal: true

Rails.application.routes.draw do
  namespace :api do
    get  'metrics_dashboard/organizations', to: 'metrics_dashboard#organizations', as: 'metrics_dashboard_organizations'
  end
end
```


### Documentation comments on actions

Precede each action with a one-line comment indicating the HTTP verb and full path, for example:

```ruby
# GET /api/metrics_dashboard/organizations
def organizations
  ...
end
```

This improves readability and helps the frontend map actions to endpoints.


### Authorization patterns

- Rhino controllers: use `policy_scope(klass)` to scope records, and `authorize(record, :action?)` for custom actions.
- API controllers: explicitly enforce access via `before_action` checks, or use Pundit’s `authorize` with hand-picked policies when appropriate.
- See the Rhino policy concepts for role-based policy composition and strong params: [Policies](https://www.rhino-project.org/docs/concepts/auth/policies).


### Best practices

- Keep controllers thin. Move complex logic to models or helpers.
- Validate all incoming params; return precise error messages and appropriate HTTP status codes.
- Always document actions with `# VERB /path` comments.
- Prefer `render json:` with explicit `status:`; avoid implicit renders for APIs.
- Use helpers for aggregations/formatting (e.g., `MetricsHelper`) and models for domain operations.
- Split long actions into smaller private methods; avoid large `index` methods when possible.


### Implementation checklist

1. Decide whether the endpoint is model-backed (Rhino) or a pure API endpoint.
2. Create the controller in `app/controllers/rhino` or `app/controllers/api` with the correct module.
3. Add routes under `scope module: 'rhino'` for Rhino controllers or `namespace :api` for API controllers.
4. Add one-line endpoint comments above each action (e.g., `# GET /api/...`).
5. Enforce authorization: `policy_scope`/`authorize` (Rhino) or explicit `before_action` checks (API).
6. Validate params and return meaningful errors with appropriate HTTP status codes.
7. Keep actions small; push business logic to helpers/models.


