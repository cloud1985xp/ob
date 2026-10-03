# Subject Management Reimplementation PRP

name: "Subject Management with Admin Domain Module and Modern UI"
description: |

## Purpose
Reimplement subject management functionality with proper domain organization, many-to-many associations, polymorphic relationships, and modern LiveView UI.

## Core Principles
1. **Context is King**: Follow existing Venus codebase patterns exactly
2. **Validation Loops**: Test migrations, compilation, and UI functionality  
3. **Information Dense**: Mirror Venus.Actresses domain module structure
4. **Progressive Success**: Database first, then domain modules, then LiveView
5. **Global rules**: Follow all rules in CLAUDE.md

---

## Goal
Re-implement subject management at /subjects with modern domain architecture, proper associations, and beautiful table-based UI with pagination, search, and CRUD operations.

## Why
- **Business value**: Better organization and management of subjects for admin users
- **Integration**: Proper domain boundaries following DDD principles like Actresses module
- **Problems solved**: Current subject management is basic and doesn't follow codebase patterns

## What
Modern subject management system with:
- Domain module organization (lib/venux/admins.ex and lib/venux/admins/*)
- Many-to-many subject ↔ site associations  
- Optional polymorphic trackable associations (Actress or Work)
- LiveView table UI with pagination (20 per page), search, edit/delete actions
- Modern styling consistent with actresses page

### Success Criteria
- [ ] Subjects accessible at /subjects with modern table UI
- [ ] Pagination works with 20 subjects per page
- [ ] Edit/Delete actions work with icons
- [ ] Many-to-many subject-site associations function
- [ ] Optional polymorphic trackable associations work
- [ ] Domain module follows Venus.Actresses pattern exactly

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://hexdocs.pm/ecto/polymorphic-associations-with-many-to-many.html
  why: Proper polymorphic association patterns - avoid two-column approach
  critical: Use join tables for many-to-many, belongs_to for polymorphic trackable

- url: https://fullstackphoenix.com/tutorials/pagination-with-phoenix-liveview  
  why: LiveView pagination patterns and Scrivener usage
  critical: Use handle_params/3 for pagination, streams for efficient rendering

- file: lib/venux/actresses.ex
  why: Exact domain module pattern to follow for lib/venux/admins.ex
  critical: Import patterns, pagination, search functions

- file: lib/venux/actresses/actress.ex  
  why: Schema organization pattern for lib/venux/admins/subject.ex
  critical: Schema structure, changeset patterns

- file: lib/venux_web/live/actresses_live/index.ex
  why: LiveView implementation pattern for subjects
  critical: mount/3, handle_event/3, handle_params/3, streams, pagination

- file: lib/venux_web/live/actresses_live/index.html.heex
  why: Modern table UI patterns and Tailwind styling
  critical: Table structure, search bar, pagination, actions

- file: lib/venux/subject.ex
  why: Current schema fields and structure to preserve
  critical: Keep existing fields, add associations

- file: lib/venux/site.ex  
  why: Current site schema for many-to-many association
  critical: Understand existing site structure
```

### Current Codebase State
```bash
lib/venux/
├── subject.ex                    # Current flat schema
├── site.ex                      # Current flat schema  
├── actresses.ex                 # PATTERN TO FOLLOW
├── actresses/
│   ├── actress.ex              # PATTERN TO FOLLOW
│   ├── actress_work.ex
│   └── actress_work_picture.ex
└── repo/
    └── subject.ex              # Current repo functions

lib/venux_web/
├── controllers/
│   ├── subject_controller.ex   # Basic controller TO REPLACE
│   └── subject_html/
│       └── index.html.heex     # Basic template TO REPLACE
└── live/
    └── actresses_live/         # PATTERN TO FOLLOW
        ├── index.ex           # PATTERN TO FOLLOW  
        └── index.html.heex    # PATTERN TO FOLLOW
```

### Desired Codebase State
```bash
lib/venux/
├── admins.ex                    # NEW: Domain module following actresses.ex
├── admins/
│   ├── subject.ex              # MOVED: From lib/venux/subject.ex
│   ├── site.ex                 # MOVED: From lib/venux/site.ex
│   └── subject_site.ex         # NEW: Join table schema
└── (remove repo/subject.ex)    # Domain module handles queries

lib/venux_web/
├── live/
│   └── subjects_live/          # NEW: Following actresses_live pattern
│       ├── index.ex           # NEW: LiveView module
│       └── index.html.heex    # NEW: Modern table template
└── (remove controllers/subject_*)  # Replace with LiveView
```

### Known Gotchas & Library Quirks
```elixir
# CRITICAL: Ecto many_to_many requires join_through table
# Pattern from actresses: many_to_many :sites, Site, join_through: "subject_sites"

# CRITICAL: Phoenix LiveView streams require DOM id
# Pattern: stream(:subjects, subjects) with id={id} in template

# CRITICAL: Domain modules use alias and import patterns  
# Pattern from actresses.ex: alias Venus.Repo, alias Venus.Admins.{Subject}

# CRITICAL: Router requires live routes instead of resources
# Pattern: live "/subjects", SubjectsLive.Index, :index

# CRITICAL: Polymorphic associations use belongs_to with foreign keys
# Pattern: belongs_to with foreign_key and define_type options
```

## Implementation Blueprint

### Database Schema Changes
```elixir
# Migration 1: Create subject_sites join table
defp change do
  create table(:subject_sites) do
    add :subject_id, references(:subjects, on_delete: :delete_all), null: false  
    add :site_id, references(:sites, on_delete: :delete_all), null: false
    timestamps()
  end
  
  create unique_index(:subject_sites, [:subject_id, :site_id])
end

# Migration 2: Update subjects table for polymorphic trackable (optional)
# trackable_type and trackable_id already exist, ensure they're nullable
```

### Domain Module Structure
```elixir
# lib/venux/admins.ex - Mirror lib/venux/actresses.ex exactly
defmodule Venus.Admins do
  import Ecto.Query
  alias Venus.Repo
  alias Venus.Admins.{Subject, Site}
  
  # Pagination and search patterns from actresses.ex
  def list_subjects(params \\ []) do
    # Search, pagination, ordering logic
  end
end

# lib/venux/admins/subject.ex - Mirror schema patterns
defmodule Venus.Admins.Subject do
  use Ecto.Schema
  import Ecto.Changeset
  
  schema "subjects" do
    # Existing fields preserved
    field :name, :string
    field :term, :string  
    field :last_identifier, :string
    field :enabled, :boolean, default: true
    
    # Polymorphic trackable association (optional)
    field :trackable_type, :string
    field :trackable_id, :integer
    
    # Many-to-many sites association
    many_to_many :sites, Venus.Admins.Site, 
      join_through: "subject_sites", 
      on_replace: :delete
      
    timestamps(type: :utc_datetime, inserted_at: :created_at)
  end
end
```

### Task List (Execute in Order)
```yaml
Task 1 - Database Migrations:
  CREATE priv/repo/migrations/*_create_subject_sites.exs:
    - Create join table with proper foreign keys and constraints
    - Add unique index on [subject_id, site_id]
  
Task 2 - Domain Module Structure:  
  CREATE lib/venux/admins.ex:
    - Mirror lib/venux/actresses.ex structure exactly
    - Implement list_subjects/1 with pagination and search
    - Add find/1, create/1, update/2, delete/1 functions
    
  CREATE lib/venux/admins/ directory:
    - Move lib/venux/subject.ex to lib/venux/admins/subject.ex
    - Move lib/venux/site.ex to lib/venux/admins/site.ex  
    - Update module names and add associations
    
  CREATE lib/venux/admins/subject_site.ex:
    - Join table schema for many-to-many association
    
Task 3 - Remove Old Files:
  DELETE lib/venux/subject.ex:
    - File moved to lib/venux/admins/subject.ex
  DELETE lib/venux/site.ex:  
    - File moved to lib/venux/admins/site.ex
  DELETE lib/venux/repo/subject.ex:
    - Functionality moved to domain module
    
Task 4 - LiveView Implementation:
  CREATE lib/venux_web/live/subjects_live/ directory:
  
  CREATE lib/venux_web/live/subjects_live/index.ex:
    - Mirror lib/venux_web/live/actresses_live/index.ex exactly
    - Implement mount/3, handle_event/3, handle_params/3
    - Use streams for subjects with pagination
    - Add search functionality
    
  CREATE lib/venux_web/live/subjects_live/index.html.heex:
    - Mirror actresses table UI patterns
    - Table with columns: Name, Term, Last Identifier, Enabled, Actions
    - Modern styling with Tailwind CSS
    - Edit/Delete action buttons with icons
    - Pagination with 20 per page
    
Task 5 - Routing Updates:
  MODIFY lib/venux_web/router.ex:
    - REMOVE: resources "/subjects", SubjectController
    - ADD: live "/subjects", SubjectsLive.Index, :index
    
Task 6 - Remove Old Controller:
  DELETE lib/venux_web/controllers/subject_controller.ex
  DELETE lib/venux_web/controllers/subject_html/ directory
  
Task 7 - Update References:
  SEARCH and UPDATE any imports of Venus.Subject to Venus.Admins.Subject
  SEARCH and UPDATE any imports of Venus.Site to Venus.Admins.Site
```

### Critical Implementation Details

#### Task 1 Pseudocode - Migration:
```elixir
# Create migration with proper constraints
def change do
  create table(:subject_sites, primary_key: false) do
    add :subject_id, references(:subjects, on_delete: :delete_all), 
        null: false, primary_key: true
    add :site_id, references(:sites, on_delete: :delete_all), 
        null: false, primary_key: true
    timestamps()
  end
  
  # Ensure unique constraint
  create unique_index(:subject_sites, [:subject_id, :site_id])
end
```

#### Task 2 Pseudocode - Domain Module:
```elixir
# Follow exact pattern from lib/venux/actresses.ex
def list_subjects(params \\ []) do
  search_query = params["search"]
  
  Subject
  |> maybe_search(search_query)  
  |> order_by(desc: :id)
  |> Repo.paginate(params)  # Uses Scrivener
end

# Add preloading for associations
def list_subjects_with_sites(params \\ []) do
  # Preload sites association for display
  list_subjects(params) |> Repo.preload(:sites)
end
```

#### Task 4 Pseudocode - LiveView:
```elixir
# Mirror actresses_live/index.ex patterns exactly
def mount(_params, _session, socket) do
  subjects_page = Admins.list_subjects(%{"page" => 1})
  
  socket = 
    socket
    |> assign(page_title: "Subject Management")
    |> assign(current_page: 1, search_query: "")
    |> assign(has_more: length(subjects_page.entries) == 20)
    |> stream(:subjects, subjects_page.entries)
    
  {:ok, socket}
end

def handle_event("search", %{"search" => %{"query" => query}}, socket) do
  # Reset stream and search - mirror actresses pattern
end

def handle_event("delete", %{"id" => id}, socket) do
  # Delete subject and update stream
end
```

## Validation Loop

### Level 1: Database & Compilation
```bash
# Run these FIRST - fix any errors before proceeding
mix ecto.migrate                 # Database migrations successful
mix compile                     # No compilation errors  
mix credo --strict              # Code quality checks

# Expected: No errors. If errors, READ the error and fix.
```

### Level 2: Domain Logic Tests  
```bash
# Test domain module functions work
iex -S mix

# Test subject listing
Venus.Admins.list_subjects()

# Test subject with sites
Venus.Admins.list_subjects() |> Venus.Repo.preload(:sites)

# Expected: No errors, proper associations loaded
```

### Level 3: LiveView Integration Test
```bash
# Start Phoenix server
mix phx.server

# Navigate to subjects page
open http://localhost:4000/subjects

# Test pagination, search, actions work
# Expected: Modern table loads with 20 subjects per page
```

## Final Validation Checklist
- [ ] Migration successful: `mix ecto.migrate` 
- [ ] No compilation errors: `mix compile`
- [ ] Code quality passes: `mix credo`
- [ ] /subjects page loads with modern table UI
- [ ] Pagination works (20 per page)
- [ ] Search functionality works
- [ ] Edit/Delete actions display with icons
- [ ] Subject-site associations load properly
- [ ] Domain module follows actresses pattern exactly

---

## Anti-Patterns to Avoid
- ❌ Don't create new UI patterns - mirror actresses exactly
- ❌ Don't skip join table constraints - use proper foreign keys
- ❌ Don't use old controller patterns - use LiveView only
- ❌ Don't ignore existing domain module patterns
- ❌ Don't hardcode pagination size - use configurable values  
- ❌ Don't forget to preload associations in listings
- ❌ Don't create custom CSS - use existing Tailwind patterns

## Confidence Score: 9/10
High confidence due to:
- Exact patterns exist in actresses module to follow
- Clear documentation references for Ecto associations  
- Step-by-step migration and validation approach
- Comprehensive context and gotchas identified
- Executable validation commands provided