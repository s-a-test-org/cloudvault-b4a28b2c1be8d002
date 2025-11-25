## Rhino Models: Authoring Guide for AI

This document explains how to create and maintain models in a Rhino-powered Rails application. It is written for AI-assisted development, with explicit structure, conventions, and examples.

Always generate models with Rails generators first:

```bash
bin/rails g model ModelName ...
```

Then open the model file and implement the sections below in the indicated order with comment headers for clarity.


### Standard file structure

Every model should be structured using explicit section comments:

- `# associations`
- `# rhino`
- `# callbacks`
- `# attributes`
- `# enums` (optional)
- `# validations`
- `# notifications` (optional)
- `# scopes` (optional)
- `# class methods` (optional)
- `# instance methods`
- `private` (then private helpers, each preceded by a short comment block explaining the method’s purpose)


### Rhino ownership and references

Rhino requires explicit ownership and references to build API schemas, authorization, and relationships.

- If the model belongs to a single clear owner, set `rhino_owner :owner_association`.
- If the model is globally owned (not scoped by another model), use `rhino_owner_global`.
- Always set `rhino_references [...]` to include the owner and any other referenced associations used by the API or frontend.
- If a model belongs to multiple parents, choose the most “valuable” or primary parent as the `rhino_owner`. This is primarily to satisfy OpenAI schema generation that expects a single owner, though any parent could work in practice. All parents should be included in `rhino_references`.


### Display name fallback

Rhino will try to use a `name` column for display. If the model has no `name` column, implement `display_name`:

```ruby
def display_name
  # Generate a human-friendly name when there is no `name` attribute
  # Example: "##{id} – #{some_attribute}" or a composition of parent names for join models
end
```


### Searchable properties

To make a model searchable, declare searchable attributes and optional relationship fields:

```ruby
rhino_search %i[column_name], { related_model: %i[title] }
```

The second argument is a hash mapping association names to arrays of their searchable attributes.


### Readable labels for properties

The frontend renders labels from column names. Override the label using:

```ruby
rhino_properties_readable_name(content: "Message")
```

See more options in the Rhino properties guide: [Properties](https://www.rhino-project.org/docs/guides/properties).


### File attachments

Use Active Storage to attach files. Reference the attachment relationship in `rhino_references`:

```ruby
has_one_attached :document_file
rhino_references [:owner, :document_file_attachment]
```

For multiple files:

```ruby
has_many_attached :images
rhino_references [:owner, :images_attachments]
```

The symbol after `has_one_attached`/`has_many_attached` should be the rendered column name for the model.


### Nested attributes

When a model manages nested records, declare nested attributes and, if needed, restrict array operations via Rhino:

```ruby
accepts_nested_attributes_for :sub_items, allow_destroy: true
rhino_properties_array sub_items: { creatable: true, updatable: true, destroyable: true }
```

Note: `allow_destroy: true` must be enabled on `accepts_nested_attributes_for` for `destroyable` to work. See details in the Rhino guide.


### Custom/synthetic attributes in API responses

Sometimes we need attributes that are not database columns but should appear in responses or be used as inputs handled by callbacks.

Use Rails `attribute` for type definition and implement getters/setters:

```ruby
attribute :computed_total, :decimal

def computed_total
  # Calculate dynamically from other fields
end

def computed_total=(value)
  # Optionally accept an input to influence state (e.g., set other attributes)
end
```

You can also restrict when a synthetic attribute can be written:

```ruby
rhino_properties_create except: :computed_total
```

For persisted computed values, prefer generated columns (see below) instead of runtime calculations.


### Properties: restricting visibility and writes

Use Rhino helpers to restrict read/write surfaces for attributes exposed by the API.

- Include specific properties only:

```ruby
rhino_properties_read   only: %i[id uid title email]
rhino_properties_create only: %i[title summary email]
rhino_properties_update only: %i[title summary]
```

- Exclude specific properties:

```ruby
rhino_properties_read  except: %i[secret_token]
rhino_properties_write except: %i[email]
```

- Restrict array operations for nested properties:

```ruby
accepts_nested_attributes_for :tags, allow_destroy: true
rhino_properties_array tags: { creatable: false, updatable: true, destroyable: true }
```


### Properties: formats and display names

- Set a specific display format (useful for front-end rendering):

```ruby
rhino_properties_format phone: :phone, categories: :join_table_simple
```

- Override readable labels for one or more properties:

```ruby
rhino_properties_readable_name title: "Name", description: "Body"
```

Reference: [Properties](https://www.rhino-project.org/docs/guides/properties)


### Tags

Enable tagging using `acts_as_taggable_on`:

```ruby
acts_as_taggable_on :tags
rhino_references %i[owner]
```

This automatically exposes the `tags` property.


### Additional backend validations and formats

Rhino integrates helpful validations and formatting. Common patterns:

- Country (using `countries` gem):

```ruby
validates :country, country: { allow_blank: true }               # optional
validates :country, country: { alpha3: true }                    # required alpha3
```

- Currency (displayed as currency in the front end):

```ruby
rhino_properties_format amount: :currency  # store as decimal
```

- IPv4:

```ruby
validates :ipv4, ipv4: { allow_blank: true }
```

- MAC address:

```ruby
validates :mac_address, mac_address: { allow_blank: true }
```

- Phone number (using `phonelib`):

```ruby
rhino_properties_format phone: :phone

before_validation :normalize_phone
validates :phone, phone: { message: "not a valid phone number", possible: true }

private
  def normalize_phone
    self.phone = Phonelib.parse(phone).full_e164.presence
  end
```


### Changing validation messages

You can override validation messages with lambdas to include contextual information:

```ruby
validates :email, presence: true, email: true, uniqueness: {
  message: lambda do |object, data|
    existing = object.class.find_by(email: data[:value])
    "has already been taken by: #{existing&.respond_to?(:display_name) ? existing.display_name : existing&.id}"
  end
}
```


### Generated columns (database-level computed attributes)

Prefer database-generated columns when the computed value should be stored and indexed:

```ruby
def change
  create_table :readings do |t|
    t.decimal :celsius
    t.virtual :fahrenheit, type: :decimal, as: 'celsius * 9 / 5 + 32', stored: true
  end
end
```

or:

```ruby
def change
  add_column :readings, :fahrenheit, :decimal, as: 'celsius * 9 / 5 + 32', stored: true, virtual: true
end
```

Rails currently does not support non-stored virtual columns.


### Read-only attributes

Attributes marked with `attr_readonly` are set on create and ignored on update:

```ruby
attr_readonly :external_id
```

Rhino will surface them as creatable but not updatable.


### Notifications (when applicable)

If the feature requires user notifications, follow this pattern. Implement only when requested by the feature requirements.

```ruby
acts_as_notifiable :users,
                   dependent_notifications: :update_group_and_destroy,
                   targets: ->(record, key) { record.notifiable_targets(key) },
                   notifiable_path: :frontend_notifiable_path,
                   printable_name: :display_name

after_create_commit :notify_on_create
after_update_commit :notify_on_update, if: -> {
  saved_change_to_attribute?(:status) || saved_change_to_attribute?(:due_on)
}

def notifiable_targets(_key)
  # Return a set of User records to notify
  watchers
end

def frontend_notifiable_path
  route_frontend
end

private
  def notify_on_create
    notify_later :users, key: "work_item.created" if watchers.any?
  end

  def notify_on_update
    notify_later :users, key: "work_item.updated" if watchers.any?
  end
```


---

## End-to-end examples

The following classes illustrate the conventions. Names are illustrative and unrelated to any existing app entities.


### Example 1: WorkItem (core model with ownership, search, validations, nested attributes, notifications)

```ruby
# frozen_string_literal: true

class WorkItem < ApplicationRecord
  # associations
  belongs_to :portfolio
  has_many :assignments, dependent: :destroy
  has_many :assignees, through: :assignments, source: :user
  has_many :uploads, dependent: :destroy
  has_many :remarks, dependent: :destroy

  accepts_nested_attributes_for :assignments, allow_destroy: true

  # rhino
  rhino_owner :portfolio
  rhino_references [ :portfolio, { assignments: [ :user ] } ]
  rhino_search %i[title], { portfolio: %i[name] }

  # callbacks
  before_create :ensure_default_status
  before_save :coerce_dates

  # attributes
  enum :status, {
    "Planned"      => "planned",
    "In Progress"  => "in_progress",
    "Review"       => "review",
    "Done"         => "done",
    "Archived"     => "archived"
  }, prefix: true

  attribute :created_from_template, :boolean, default: false

  # validations
  validates :title, presence: true
  validates :summary, presence: true
  validate  :require_due_or_ongoing
  validate  :require_at_least_one_assignee, if: -> { created_from_template.blank? }

  # notifications (optional; include only when required)
  acts_as_notifiable :users,
                     dependent_notifications: :update_group_and_destroy,
                     targets: ->(record, key) { record.notifiable_targets(key) },
                     notifiable_path: :frontend_notifiable_path,
                     printable_name: :title
  after_create_commit :notify_assignees_on_create
  after_update_commit :notify_assignees_on_update, if: -> {
    saved_change_to_status? || saved_change_to_due_on? || saved_change_to_ongoing?
  }

  def notifiable_targets(_key)
    assignees
  end

  def frontend_notifiable_path
    route_frontend
  end

  # instance methods
  def display_name
    title
  end

  private
    # validations
    def require_at_least_one_assignee
      has_keepable = assignments.any? { |a| !a.marked_for_destruction? }
      errors.add(:base, "at least one user must be assigned") unless has_keepable
    end

    def require_due_or_ongoing
      if !ongoing && due_on.blank?
        errors.add(:due_on, "must exist or set item as ongoing")
      end
      if ongoing && due_on.present?
        errors.add(:due_on, "cannot be present when ongoing")
      end
      if !ongoing && due_on.present? && due_on < Time.zone.today && %w[planned in_progress review].include?(status)
        errors.add(:due_on, "cannot be in the past")
      end
    end

    # callbacks
    def ensure_default_status
      self.status ||= "planned"
    end

    def coerce_dates
      self.review_on = nil if due_on.present? && review_on.present? && ongoing.nil?
    end

    # notifications
    def notify_assignees_on_create
      notify_later :users, key: "work_item.created" if assignees.any?
    end

    def notify_assignees_on_update
      notify_later :users, key: "work_item.updated" if assignees.any?
    end
end
```


### Example 2: Upload (file attachment + search)

```ruby
# frozen_string_literal: true

class Upload < ApplicationRecord
  # associations
  belongs_to :work_item
  has_one_attached :file_blob

  # rhino
  rhino_owner :work_item
  rhino_references [ :work_item, :file_blob_attachment ]
  rhino_search %i[file_name], { work_item: %i[title] }

  # callbacks
  before_save :derive_file_name
  before_save :stamp_uploaded_at

  # instance methods
  def display_name
    file_name.presence || "Upload ##{id}"
  end

  private
    # sets file_name from the blob
    def derive_file_name
      self.file_name = file_blob.filename.to_s if file_blob.attached?
    end

    # stores upload timestamp
    def stamp_uploaded_at
      self.uploaded_at ||= Time.zone.now
    end
end
```


### Example 3: Remark (alternate label + display_name without `name` column)

```ruby
# frozen_string_literal: true

class Remark < ApplicationRecord
  # associations
  belongs_to :work_item
  belongs_to :author, class_name: "User"

  # rhino
  rhino_owner :work_item
  rhino_references [ :work_item, :author ]
  rhino_properties_readable_name(body: "Message")

  # instance methods
  def display_name
    "Remark by #{author&.name || "Unknown"}" 
  end
end
```


### Example 4: Global model (no parent)

```ruby
# frozen_string_literal: true

class GlobalSetting < ApplicationRecord
  # rhino
  rhino_owner_global

  # validations
  validates :key, presence: true, uniqueness: true

  # instance methods
  def display_name
    key
  end
end
```


### Example 5: Join model (pivot table)

```ruby
# frozen_string_literal: true

class Membership < ApplicationRecord
  # associations
  belongs_to :team
  belongs_to :user

  # rhino
  rhino_owner :team  # choose a single owner, include others in references
  rhino_references [ :team, :user ]

  # instance methods
  def display_name
    "#{team&.name || "Team"} – #{user&.name || "User"}"
  end
end
```


---

## Implementation checklist for new models

1. Generate the model with Rails and run migrations.
2. Add `# associations` and declare all `belongs_to`, `has_many`, etc.
3. Add `# rhino`: set one of `rhino_owner :owner` or `rhino_owner_global`; add `rhino_references [...]`.
4. If searchable, add `rhino_search` including relationship fields if needed.
5. If using attachments, declare `has_one_attached`/`has_many_attached` and reference attachments in `rhino_references`.
6. If handling nested records, declare `accepts_nested_attributes_for` and `rhino_properties_array` as needed.
7. Add `# attributes` and `# enums` (e.g., `enum :status, ...`).
8. Add `# validations` including any additional validators (phone, country, etc.).
9. Add `# callbacks` for defaults and coercions.
10. Implement `display_name` if no `name` column exists, or for join models combine parent names.
11. If custom properties are needed in the API, define synthetic attributes via `attribute :...` and restrict writes with `rhino_properties_*` when appropriate.
12. If notifications are part of the feature, add the notification section and hooks.
13. Consider generated columns if values should be stored and indexed.
14. Use `rhino_properties_readable_name` for UI label overrides.


## Reference

- Rhino Properties Guide: [https://www.rhino-project.org/docs/guides/properties](https://www.rhino-project.org/docs/guides/properties)


