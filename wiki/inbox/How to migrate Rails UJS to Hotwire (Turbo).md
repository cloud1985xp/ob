---
title: "How to migrate Rails UJS to Hotwire (Turbo)"
source: "https://dev.to/thomasvanholder/how-to-migrate-rails-ujs-to-hotwire-turbo-hdh"
author:
  - "[[thomasvanholder]]"
published: 2021-12-11
created: 2026-05-05
description: "See Hotwire handbook   Update hotwired/turbo-rails Remove rails/ujs Replace link_to delete Replace... Tagged with rails, hotwire, tailwindcss, tutorial."
tags:
  - "clippings"
---
See [Hotwire handbook](https://turbo.hotwired.dev/handbook/drive#performing-visits-with-a-different-method)

## 1\. Update hotwired/turbo-rails

In **package.json**, update `@hotwired/turbo-rails` package to 7.1.0 (or greater)

```
"@hotwired/turbo-rails": "^7.1.0"
```

---

## 2\. Remove rails/ujs

- From the package.json file
```
yarn remove @rails/ujs
```
- From the application.js entry point
```
require("@rails/ujs").start() // remove this line
```

---

## 3\. Replace link\_to delete

`method: :delete`  
becomes  
`data: {turbo_method: :delete}`

```
<%# Old %>
<% link_to "Destroy", task_path(@task), method: :delete %>

<%# New %>
<% link_to "Destroy", task_path(@task), data: {turbo_method: :delete} %>
```

---

## 4\. Replace link\_to data confirm

`data: {confirm: 'Are you sure?'}`  
becomes  
`data: {turbo_confirm: 'Are you sure?'}`

```
<%# Old %>
<% link_to "Destroy", task_path(@task), method: :delete, data: {confirm: 'Are you sure?'} %>

<%# New %>
<% link_to "Destroy", task_path(@task), data: {turbo_method: :delete, turbo_confirm: 'Are you sure?'} %>
```

---

## 5\. Replace button data disable with

In Rails UJS, a form's submit button can be disabled and its text replaced by adding the data disable\_with attribute.

```
<%= f.button "Search", data: { disable_with: "Searching..."} %>
```

With Turbo, you can set the text content based on the parent's button disabled status. [Source](https://github.com/hotwired/turbo/pull/386)

```
button             .show-when-disabled { display: none; }
button[disabled]   .show-when-disabled { display: initial; }

button             .show-when-enabled { display: initial; }
button[disabled    .show-when-enabled { display: none; }
```
```
<button>
  <span class="show-when-enabled">Submit</span>
  <span class="show-when-disabled">Submitting...</span>
</button>
```

Or if you use Tailwind, you can leverage Tailwind's [group](https://tailwindcss.com/docs/hover-focus-and-other-states#styling-based-on-parent-state) and [disabled](https://tailwindcss.com/docs/hover-focus-and-other-states#disabled) status to create a similar effect.

```
<%= f.button class: "group" do %>
   <span class="group-disabled:hidden">Search</span>
   <span class="hidden group-disabled:block group-disabled:cursor-wait">Searching...</span>
<% end %>
```

---

## 6\. Set status response in controller

Respond with a [303 status code](http://www.railsstatuscodes.com/see_other.html)

```
class TasksController
 ...

  def destroy
    @task = Task.find(params[:id])
    @task.destroy
    redirect_to tasks_path, info: "Task deleted", status: :see_other
  end 
end
```[Sonar](https://dev.to/sonar)Promoted

[![State of Code Developer Survey report](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fucarecdn.com%2F2f2ce9b0-68e0-48a1-bf3e-46c08831a9be%2F)](https://www.sonarsource.com/sem/the-state-of-code/developer-survey-report/?utm_medium=paid&utm_source=dev&utm_campaign=ss-state-of-code-developer-survey26&utm_content=report-devsurvey-banner-x-2&utm_term=ww-all-x&s_category=Paid&s_source=Paid+Social&s_origin=dev&bb=260505)

## State of Code Developer Survey report

Did you know 96% of developers don't fully trust that AI-generated code is functionally correct, yet only 48% always check it before committing? Check out Sonar's new report on the real-world impact of AI on development teams.