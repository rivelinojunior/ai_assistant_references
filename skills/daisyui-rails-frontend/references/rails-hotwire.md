# Rails + Hotwire Patterns

Reference for deeper Turbo and Stimulus implementation details. Read this when the main skill's quick examples aren't enough.

---

## Turbo Frames

### Lazy Loading

```erb
<%# Load content on demand — server renders it when the frame scrolls into view or page loads %>
<turbo-frame id="recent-meals" loading="lazy" src="<%= recent_meals_path %>">
  <div class="loading loading-dots loading-sm"></div>
</turbo-frame>
```

### Targeting Outside the Frame

```erb
<%# Link that navigates the whole page, not just the frame %>
<turbo-frame id="meal-card">
  <%= link_to "View full plan", plan_path(@plan), data: { turbo_frame: "_top" } %>
</turbo-frame>
```

### Form Inside a Frame

```erb
<turbo-frame id="edit-meal-<%= @meal.id %>">
  <%= form_with model: @meal do |f| %>
    <%# On success, controller responds with turbo_stream or redirects — both work %>
    ...
  <% end %>
</turbo-frame>
```

---

## Turbo Streams

### Response Patterns

```ruby
# In controller — respond with multiple stream actions
def update
  if @meal.update(meal_params)
    respond_to do |format|
      format.turbo_stream do
        render turbo_stream: [
          turbo_stream.replace("meal-#{@meal.id}", partial: "meals/meal", locals: { meal: @meal }),
          turbo_stream.prepend("flash-container", partial: "shared/flash",
                               locals: { type: :notice, message: "Meal updated" })
        ]
      end
      format.html { redirect_to @meal }
    end
  else
    respond_to do |format|
      format.turbo_stream do
        render turbo_stream: turbo_stream.replace(
          "edit-meal-form",
          partial: "meals/form",
          locals: { meal: @meal }
        )
      end
      format.html { render :edit, status: :unprocessable_entity }
    end
  end
end
```

### Stream Actions

| Action | Effect |
|---|---|
| `turbo_stream.replace(id, ...)` | Replace the element with matching id |
| `turbo_stream.update(id, ...)` | Replace the element's *content* (keep wrapper) |
| `turbo_stream.append(id, ...)` | Append to element's children |
| `turbo_stream.prepend(id, ...)` | Prepend to element's children |
| `turbo_stream.remove(id)` | Remove the element |
| `turbo_stream.after(id, ...)` | Insert after the element |
| `turbo_stream.before(id, ...)` | Insert before the element |

### Real-time via ActionCable

```erb
<%# Subscribe to a stream in the view %>
<%= turbo_stream_from @meal_plan %>
<div id="meal-plan-<%= @meal_plan.id %>">
  ...
</div>
```

```ruby
# In the model or job — broadcast changes
MealPlan.after_update_commit do
  broadcast_replace_to self,
    target: "meal-plan-#{id}",
    partial: "meal_plans/meal_plan",
    locals: { meal_plan: self }
end
```

---

## Stimulus

### Controller Lifecycle

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  // Called once when element enters DOM
  connect() {}

  // Called when element leaves DOM
  disconnect() {}

  // Called when a value changes (if using values API)
  // [valueName]ValueChanged(value, previousValue) {}
}
```

### Values API

```javascript
static values = {
  url: String,
  count: { type: Number, default: 0 },
  open: { type: Boolean, default: false },
  items: Array,
  config: Object
}

// Access: this.urlValue, this.countValue, etc.
// Observe: countValueChanged(value, previousValue) {}
```

```erb
<div data-controller="pagination"
     data-pagination-url-value="<%= api_meals_path %>"
     data-pagination-count-value="<%= @meals.count %>">
```

### Targets API

```javascript
static targets = ["input", "output", "submit"]

// Access: this.inputTarget (first), this.inputTargets (all), this.hasInputTarget
```

```erb
<div data-controller="search">
  <input data-search-target="input" type="text" />
  <div data-search-target="output"></div>
</div>
```

### Actions

```erb
<%# Shorthand: event->controller#method %>
<button data-action="click->modal#open">Open</button>

<%# Multiple events %>
<input data-action="input->search#query keydown.enter->search#submit" />

<%# Custom events from Turbo %>
<div data-action="turbo:frame-load->spinner#hide">
```

### Outlets (cross-controller communication)

```javascript
// parent_controller.js
static outlets = ["child"]

someAction() {
  this.childOutlets.forEach(outlet => outlet.update())
}
```

```erb
<div data-controller="parent" data-parent-child-outlet="#child-element">
  <div id="child-element" data-controller="child">...</div>
</div>
```

### Fetch API Pattern in Stimulus

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { url: String }

  async load() {
    const response = await fetch(this.urlValue, {
      headers: { Accept: "text/vnd.turbo-stream.html" }
    })

    if (response.ok) {
      const html = await response.text()
      Turbo.renderStreamMessage(html)
    }
  }
}
```

---

## Common Patterns

### Inline Edit

```erb
<%# View mode %>
<div id="meal-name-<%= meal.id %>">
  <turbo-frame id="meal-name-frame-<%= meal.id %>">
    <span><%= meal.name %></span>
    <%= link_to "Edit", edit_meal_path(meal), data: { turbo_frame: "meal-name-frame-#{meal.id}" } %>
  </turbo-frame>
</div>

<%# Edit partial (edit.html.erb) — loaded into the frame %>
<turbo-frame id="meal-name-frame-<%= @meal.id %>">
  <%= form_with model: @meal do |f| %>
    <%= f.text_field :name, class: "input input-bordered input-sm" %>
    <%= f.submit "Save", class: "btn btn-primary btn-sm" %>
    <%= link_to "Cancel", meal_path(@meal), data: { turbo_frame: "meal-name-frame-#{@meal.id}" } %>
  <% end %>
</turbo-frame>
```

### Optimistic UI Toggle

```javascript
// toggle_controller.js — immediate visual feedback, server confirms
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["icon"]
  static values = { url: String, active: Boolean }

  async toggle() {
    // Optimistic update
    this.activeValue = !this.activeValue

    const response = await fetch(this.urlValue, {
      method: "PATCH",
      headers: {
        "X-CSRF-Token": document.querySelector('meta[name="csrf-token"]').content,
        "Content-Type": "application/json"
      },
      body: JSON.stringify({ active: this.activeValue })
    })

    if (!response.ok) {
      // Revert on failure
      this.activeValue = !this.activeValue
    }
  }

  activeValueChanged() {
    this.iconTarget.dataset.active = this.activeValue
  }
}
```

### Loading State on Form Submit

```javascript
// form_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["submit"]

  submit() {
    this.submitTarget.disabled = true
    this.submitTarget.innerHTML =
      '<span class="loading loading-spinner loading-xs"></span> Saving...'
  }
}
```

```erb
<%= form_with model: @resource, data: { controller: "form", action: "submit->form#submit" } do |f| %>
  ...
  <%= f.submit "Save", data: { form_target: "submit" },
      class: "btn btn-primary active:scale-95 transition-all duration-150" %>
<% end %>
```
