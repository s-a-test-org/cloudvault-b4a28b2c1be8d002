## Notifications: Authoring Guide for AI

This document explains how to implement user-facing notifications for a Rhino-powered Rails app. For model-side setup (acts_as_notifiable, targets, and hooks) do not repeat code here—refer to the conventions and examples in ./models.md.


### Where templates live

Notification email templates are ERB views located under:

```
/app/views/activity_notification/mailer/<target_plural>/<resource>/<event>.html.erb
```

Examples:

- When notifying users about a created work item:
  - Key: `work_item.created`
  - Target: `:users`
  - Template path: `/app/views/activity_notification/mailer/users/work_item/created.html.erb`

If you notify a different audience, the `<target_plural>` segment changes accordingly, e.g. `/app/views/activity_notification/mailer/admins/...` for `:admins`.


### Triggering a notification from the model

In your model’s notifications section, enqueue delivery with a `notify_later` call. The `key` maps directly to the `<resource>/<event>` segments in the template path. See ./models.md for the full model setup.

```ruby
# notifications
after_create_commit :notify_assignees_on_create

private
  def notify_assignees_on_create
    # Enqueues delivery to users using the template at:
    # /app/views/activity_notification/mailer/users/work_item/created.html.erb
    notify_later :users, key: "work_item.created" if assignees.any?
  end
```


### Template example (HTML)

Templates receive a `@notification` object and the `@target` record (the recipient). The notifiable domain record is accessible via `@notification.notifiable`. Keep content concise and action-oriented.

```erb
<h3>New Work Item</h3>

<p>Hi <%= @target.name %>,</p>

<p>A new work item has been created and assigned to you.</p>

<p>
  <span>
    <%= link_to "Open item", @notification.notifiable.frontend_url %>
  </span>
  <!-- Alternatively, if you expose a path helper from the model: -->
  <!-- <%= link_to "Open item", @notification.notifiable.frontend_notifiable_path %> -->
  <!-- See ./models.md for guidance. -->
  
</p>

<p>Thank you!</p>

<hr/>
```

Optionally, provide a plaintext counterpart alongside the HTML template:

```
/app/views/activity_notification/mailer/users/work_item/created.text.erb
```


### Mapping keys to paths

- Key format: `<resource>.<event>`
- Example: `comment.updated` → `/app/views/activity_notification/mailer/users/comment/updated.html.erb`
- The first segment of the path after `mailer/` is the pluralized target symbol used in `notify_later` (e.g., `:users`, `:admins`).


### Testing and delivery

- `notify_later` enqueues background delivery; use `notify_now` for immediate sending (synchronous in-process), typically only in tests.
- Ensure your background processor is running in development for `notify_later` to deliver emails.


### Implementation checklist

1. In the model, define the recipients and triggers (see ./models.md for `acts_as_notifiable`, targets, and hooks).
2. Choose a `key` like `resource.event` that mirrors your folder structure.
3. Create the corresponding HTML (and optional text) template at `/app/views/activity_notification/mailer/<target_plural>/<resource>/<event>.*.erb`.
4. In the template, use `@target` for the recipient and `@notification.notifiable` for the domain record; link to `frontend_url` or `frontend_notifiable_path` exposed by the model.
5. Verify delivery in development with a running job processor; write a basic mailer preview or request spec if needed.


