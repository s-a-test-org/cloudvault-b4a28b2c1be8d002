## Policies: Authoring Guide for AI

This document explains how to design and implement authorization policies for a Rhino-powered Rails application. Policies in Rhino are built on Pundit and integrate with Rhino roles, scopes, and strong parameters. For related model conventions that policies may depend on (e.g., properties driving strong params), see ./models.md. Reference: [Rhino Policies](https://www.rhino-project.org/docs/concepts/auth/policies).


### Roles and naming conventions

- Roles are records in the database (e.g., `Role` model). The `admin` role is required. Additional roles can be defined (e.g., `manager`, `editor`).
- Policies are named by combining the role and resource name: `<role>_<resource>_policy.rb`.
  - Example: `manager_blog_policy.rb` for a `Blog` resource and a `manager` role.
- Rhino resolves composed policies by role and resource, falling back to role-only policies when needed. See the Rhino docs for details on composition and defaults.


### Base policies to inherit from

Always inherit from one of these base classes:

- `::Rhino::ViewerPolicy`: read-only access (index?, show?)
- `::Rhino::AdminPolicy`: full CRUD (index?, show?, create?, update?, destroy?)

Pick the closest parent that matches the default capabilities you want; refine individual query methods as needed.


### Always implement a Scope

Every policy must define a `Scope` that returns only the records the `auth_owner` (current signed-in user) is allowed to see.

```ruby
# frozen_string_literal: true

class BlueprintPolicy < ::Rhino::AdminPolicy
  class Scope < ::Rhino::AdminPolicy::Scope
    def resolve
      # Example: show blueprints owned by the auth_owner's portfolios, and global ones
      scope.where(portfolio_id: [ auth_owner.portfolios, nil ])
    end
  end
end
```

Guideline: scope narrowly to what the user truly needs. Avoid returning unrelated tenant or organization data.


### Using auth_owner and record checks

`auth_owner` is the current signed-in user. `record` is the resource instance being authorized. Gate actions with `authorize_action(condition)` so that audit and policy composition are respected.

```ruby
# frozen_string_literal: true

class VendorContactPolicy < ::Rhino::AdminPolicy
  def index?
    authorize_action(true)
  end

  def show?
    authorize_action(record.tenant == auth_owner)
  end

  def create?
    authorize_action(true)
  end

  def update?
    authorize_action(record.active?)
  end

  def destroy?
    authorize_action(record.active?)
  end

  def permitted_attributes_for_create
    %i[name service phone email notes organization]
  end

  def permitted_attributes_for_update
    %i[name service phone email notes]
  end

  class Scope < ::Rhino::AdminPolicy::Scope
    def resolve
      scope.where(organization: auth_owner.organization)
    end
  end
end
```

Notes:
- Use `authorize_action(...)` wrappers in each query method to keep consistent behavior and logging.


### User-facing example with role-sensitive logic

Policies can branch on the `auth_owner` role/state to compute access. Keep Scopes tight and mirror logic in query methods as needed.

```ruby
# frozen_string_literal: true

class MemberPolicy < ::Rhino::ViewerPolicy
  def index?
    authorize_action(true)
  end

  def show?
    if auth_owner.is_manager?
      authorize_action(auth_owner.organization.members.include?(record))
    else
      authorize_action(record == auth_owner)
    end
  end

  def create?
    authorize_action(false)
  end

  def update?
    authorize_action(record == auth_owner)
  end

  def destroy?
    authorize_action(false)
  end

  def permitted_attributes_for_update
    %i[name email avatar]
  end

  def permitted_attributes_for_show
    %i[id name email avatar]
  end

  class Scope < ::Rhino::ViewerPolicy::Scope
    def resolve
      if auth_owner.is_manager?
        scope.joins(users_roles: :organization).where(users_roles: { organization_id: auth_owner.organization.id })
      else
        scope.where(id: auth_owner.id)
      end
    end
  end
end
```


### Strong parameters via policies

Policies can control permitted attributes for different actions. By default, Rhino derives parameters from model properties (`rhino_properties_read`, `rhino_properties_create`, `rhino_properties_update`). Override to customize per-role/resource behavior.

```ruby
def permitted_attributes_for_show
  %w[only_this_param]
end
```

For broader context on properties and how they feed strong params, see ./models.md and the Rhino guide: [Rhino Policies](https://www.rhino-project.org/docs/concepts/auth/policies).


### Generating policies

Use the Rhino generator to scaffold a role-based policy:

```bash
bin/rails g rhino:policy ManagerBlog
```

Change the base policy if needed:

```bash
bin/rails g rhino:policy ManagerBlog --parent=Rhino::AdminPolicy
```


### Best practices

- Prefer the least-privileged base policy, then open up actions explicitly.
- Always implement Scope; do not rely solely on query methods.
- Ensure cross-tenant/organization boundaries are enforced in both Scope and action methods.
- Keep permitted attributes minimal and role-appropriate.


