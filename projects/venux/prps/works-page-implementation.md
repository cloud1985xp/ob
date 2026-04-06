name: "Works Page Implementation - Context-Rich PRP"
description: |

## Purpose
Implement a comprehensive /works page with two-column layout, infinite scroll, and interactive sidebar for work details display.

## Core Principles
1. **Context is King**: Include ALL necessary documentation, examples, and caveats
2. **Validation Loops**: Provide executable tests/lints the AI can run and fix
3. **Information Dense**: Use keywords and patterns from the codebase
4. **Progressive Success**: Start simple, validate, then enhance
5. **Global rules**: Be sure to follow all rules in CLAUDE.md

---

## Goal
Build a modern /works page with two-column layout (6:4 ratio) featuring an infinite scroll works list on the left and an interactive sidebar on the right that displays work details when a work is selected. Include actress profile display, rating charts, and photo galleries.

## Why
- **Business value**: Showcase all works in an organized, browsable interface
- **User experience**: Netflix-like browsing with detailed work information
- **Integration**: Connects works to actress profiles and photo galleries
- **Problems solved**: Provides comprehensive work discovery and detailed viewing

## What
**User-visible behavior:**
- Two-column responsive layout with 6:4 width ratio
- Left column: infinite scroll grid of work cards (30 per page)
- Right column: initially empty, populates when work is clicked
- Work sidebar includes: actress profile, work details, ratings, photo gallery
- Mobile responsive with sidebar overlay

**Technical requirements:**
- Reuse existing WorkCard and PhotoCard components
- Create new ActressProfile component
- Implement infinite scroll with Phoenix LiveView streams
- Add work selection state management
- Include rating/scoring display system

### Success Criteria
- [ ] Works page loads with infinite scroll functionality
- [ ] Work selection updates sidebar without page reload
- [ ] Actress profile component displays correctly
- [ ] Photo gallery shows work pictures in grid layout
- [ ] Responsive design works on mobile devices
- [ ] All existing components integrate seamlessly

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#stream/3
  why: Stream usage for infinite scroll pagination
  critical: Use `at: -1` for appending new items

- url: https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#assign/3
  why: State management for sidebar selection
  critical: Update assigns atomically to prevent race conditions

- url: https://hexdocs.pm/ecto/Ecto.html#preload/3
  why: Preloading associations for efficient data access
  critical: Preload actress and pictures for sidebar display

- url: https://tailwindcss.com/docs/grid-template-columns
  why: CSS Grid for 6:4 column ratio layout
  critical: Use `grid-cols-10` with `col-span-6` and `col-span-4`

- file: /Users/aaron.kuo/projects/venux/lib/venux_web/live/actresses_live/index.ex
  why: Infinite scroll pattern to replicate exactly
  critical: Mount, handle_event("load-more"), stream usage

- file: /Users/aaron.kuo/projects/venux/lib/venux/actresses.ex
  why: Context module pattern for Venus.Works
  critical: list_actresses/1, maybe_search/2, pagination patterns

- file: /Users/aaron.kuo/projects/venux/lib/venux_web/components/work_card.ex
  why: Existing work card component to reuse
  critical: Add phx-click handler for work selection

- file: /Users/aaron.kuo/projects/venux/lib/venux_web/components/photo_card.ex
  why: Photo grid display pattern for work pictures
  critical: Grid layout and lazy image loading patterns

- file: /Users/aaron.kuo/projects/venux/lib/venux_web/components/actress_grid_card.ex
  why: Reference for ActressProfile component design
  critical: Avatar, stats, gradient backgrounds, responsive design
```

### Current Codebase tree
```bash
lib/
├── venux/
│   ├── actresses/
│   │   ├── actress_work.ex           # Work schema (needs associations)
│   │   └── actress_work_picture.ex   # Picture schema (has work_id)
│   └── actresses.ex                  # Context pattern to mirror
├── venux_web/
│   ├── components/
│   │   ├── work_card.ex             # Ready to reuse
│   │   ├── photo_card.ex            # Ready for picture grid
│   │   └── actress_grid_card.ex     # Pattern for profile component
│   ├── live/
│   │   └── actresses_live/
│   │       └── index.ex             # Infinite scroll pattern
│   └── router.ex                    # Routes already defined
```

### Desired Codebase tree with files to be added
```bash
lib/
├── venux/
│   ├── works.ex                     # NEW: Context module for data access
│   └── actresses/
│       ├── actress_work.ex          # MODIFY: Add associations
│       └── actress_work_picture.ex  # Already exists
├── venux_web/
│   ├── components/
│   │   └── actress_profile.ex       # NEW: Reusable actress profile
│   └── live/
│       └── works_live/
│           ├── index.ex             # NEW: Main LiveView
│           └── index.html.heex      # NEW: Template
```

### Known Gotchas & Library Quirks
```elixir
# CRITICAL: Phoenix LiveView requires verified routes import
use VenusWeb, :verified_routes

# CRITICAL: Stream updates must use proper patterns
socket = stream(:works, new_entries, at: -1)  # Append to end

# CRITICAL: Association preloading for efficiency
works = Repo.preload(works, [:actress, :pictures])

# CRITICAL: Two-column state requires careful assign management
socket = assign(socket, selected_work: work, selected_work_id: work.id)

# GOTCHA: WorkCard component needs click handler modification
phx-click="select-work" phx-value-work_id={@work.id}

# GOTCHA: Mobile responsive requires conditional CSS classes
class={["lg:block", if(@show_sidebar, do: "block", else: "hidden")]}

# PATTERN: Netflix-like styling consistency
bg-gray-900, from-red-500 to-pink-500, backdrop-blur-sm
```

## Implementation Blueprint

### Data models and structure
```elixir
# Venus.Works context module (new)
defmodule Venus.Works do
  # Mirror Venus.Actresses patterns exactly
  def list_works(params) do
    ActressWork
    |> maybe_search(params["search"])
    |> order_by(desc: :published_on)  # or popularity metric
    |> preload(:actress)
    |> Repo.paginate(params)
  end

  def get_work_with_details(id) do
    # For sidebar display
    ActressWork
    |> Repo.get(id)
    |> Repo.preload([:actress, :pictures])
  end
end

# ActressWork schema associations (modify existing)
defmodule Venus.Actresses.ActressWork do
  schema "actress_works" do
    belongs_to :actress, Venus.Actresses.Actress
    has_many :pictures, Venus.Actresses.ActressWorkPicture, foreign_key: :work_id
    # ... existing fields
  end
end
```

### List of tasks to be completed in order

```yaml
Task 1 - Create Venus.Works Context Module:
  CREATE lib/venus/works.ex:
    - MIRROR pattern from: lib/venus/actresses.ex
    - INCLUDE functions: list_works/1, get_work_with_details/1, search_works/2
    - USE Repo.paginate for pagination (existing pattern)
    - ADD proper preloading with [:actress, :pictures]

Task 2 - Update ActressWork Schema Associations:
  MODIFY lib/venus/actresses/actress_work.ex:
    - ADD belongs_to :actress association
    - ADD has_many :pictures association
    - KEEP existing field definitions intact

Task 3 - Create ActressProfile Component:
  CREATE lib/venus_web/components/actress_profile.ex:
    - MIRROR design from: actress_grid_card.ex
    - INCLUDE attrs: actress (required), class (optional)
    - ADD cover background, avatar, name, stats, description
    - USE lazy_img component for images
    - APPLY Netflix-like styling (gray-900, red/pink gradients)

Task 4 - Create WorksLive.Index LiveView:
  CREATE lib/venus_web/live/works_live/index.ex:
    - MIRROR structure from: actresses_live/index.ex
    - INCLUDE mount/3 with initial works loading
    - ADD handle_event("load-more") for infinite scroll
    - ADD handle_event("select-work") for sidebar updates
    - MANAGE state: works stream, selected_work, current_page, has_more

Task 5 - Create Works Index Template:
  CREATE lib/venus_web/live/works_live/index.html.heex:
    - USE grid-cols-10 with col-span-6 and col-span-4 for 6:4 ratio
    - LEFT column: infinite scroll works grid with WorkCard components
    - RIGHT column: conditional sidebar with work details
    - ADD responsive mobile overlay (pattern from actresses show)
    - INCLUDE photo gallery using PhotoCard components

Task 6 - Add Rating Display System:
  MODIFY works sidebar in template:
    - CREATE horizontal bar charts for ratings
    - USE mock categories: Story, Acting, Cinematography
    - IMPLEMENT with Tailwind width percentages and gradients
    - ADD JSON field to work schema for rating data (future)

Task 7 - Implement Action Handlers:
  ADD to WorksLive.Index:
    - handle_event("like-work") for work actions
    - handle_event("share-work") for social features
    - handle_event("close-sidebar") to clear selection
    - FOLLOW existing event patterns from photo_card.ex

Task 8 - Testing and Validation:
  CREATE test files for each module:
    - test/venus/works_test.exs (context tests)
    - test/venus_web/live/works_live/index_test.exs (LiveView tests)
    - test/venus_web/components/actress_profile_test.exs (component tests)
```

### Task Implementation Details

```elixir
# Task 1 - Venus.Works Context Pseudocode
defmodule Venus.Works do
  import Ecto.Query
  alias Venus.Repo
  alias Venus.Actresses.ActressWork

  def list_works(params \\ %{}) do
    # PATTERN: Mirror actresses.ex exactly
    base_query = from w in ActressWork,
      order_by: [desc: w.published_on],
      preload: [:actress]
    
    base_query
    |> maybe_search(params["search"])
    |> Repo.paginate(params)
  end

  # CRITICAL: Efficient preloading for sidebar
  def get_work_with_details(id) when is_binary(id) do
    from(w in ActressWork,
      where: w.id == ^id,
      preload: [:actress, :pictures]
    )
    |> Repo.one()
  end

  # PATTERN: Search functionality (optional)
  defp maybe_search(query, nil), do: query
  defp maybe_search(query, ""), do: query
  defp maybe_search(query, search_term) do
    search_pattern = "%#{search_term}%"
    where(query, [w], ilike(w.title, ^search_pattern) or ilike(w.description, ^search_pattern))
  end
end

# Task 4 - WorksLive.Index Pseudocode
defmodule VenusWeb.WorksLive.Index do
  use VenusWeb, :live_view
  alias Venus.Works

  def mount(_params, _session, socket) do
    # PATTERN: Mirror actresses index exactly
    works_page = Works.list_works(%{"page" => 1})
    
    socket = 
      socket
      |> assign(page_title: "Browse Works")
      |> assign(current_page: 1)
      |> assign(has_more: length(works_page.entries) == 30)
      |> assign(loading: false)
      |> assign(selected_work: nil)
      |> assign(selected_work_id: nil)
      |> stream(:works, works_page.entries)

    {:ok, socket}
  end

  # PATTERN: Infinite scroll (existing)
  def handle_event("load-more", _params, socket) do
    if socket.assigns.has_more and not socket.assigns.loading do
      next_page = socket.assigns.current_page + 1
      # Load and append new works
    end
  end

  # NEW: Work selection for sidebar
  def handle_event("select-work", %{"work_id" => work_id}, socket) do
    work_details = Works.get_work_with_details(work_id)
    
    socket = 
      socket
      |> assign(selected_work: work_details)
      |> assign(selected_work_id: work_id)
    
    {:noreply, socket}
  end
end
```

### Integration Points
```yaml
ROUTES:
  - already defined: live "/works", WorksLive.Index, :index
  - pattern: Follow existing LiveView routing

NAVIGATION:
  - sidebar link: Already exists in shared_sidebar.ex
  - active state: String.starts_with?(@current_path, "/works")

COMPONENTS:
  - WorkCard: Add phx-click="select-work" phx-value-work_id={@work.id}
  - PhotoCard: Use in work picture grid (no changes needed)
  - ActressProfile: New component following existing patterns

STYLING:
  - theme: Netflix-like dark (bg-gray-900, red/pink gradients)
  - responsive: Mobile sidebar overlay pattern
  - layout: CSS Grid for column ratios
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Run these FIRST - fix any errors before proceeding
mix compile                          # Compilation check
mix credo --strict                   # Static analysis
mix format                           # Code formatting

# Expected: No errors. If errors, READ the error and fix.
```

### Level 2: Unit Tests
```elixir
# CREATE test/venus/works_test.exs
defmodule Venus.WorksTest do
  use Venus.DataCase
  alias Venus.Works

  test "list_works/1 returns paginated works" do
    works_page = Works.list_works(%{"page" => 1})
    assert %Scrivener.Page{} = works_page
  end

  test "get_work_with_details/1 preloads associations" do
    work = Works.get_work_with_details("1")
    assert %Ecto.Association.NotLoaded{} != work.actress
    assert %Ecto.Association.NotLoaded{} != work.pictures
  end
end

# CREATE test/venus_web/live/works_live/index_test.exs
defmodule VenusWeb.WorksLive.IndexTest do
  use VenusWeb.ConnCase
  import Phoenix.LiveViewTest

  test "mount displays works grid", %{conn: conn} do
    {:ok, view, html} = live(conn, ~p"/works")
    assert html =~ "Browse Works"
    assert has_element?(view, "[data-testid='works-grid']")
  end

  test "selecting work updates sidebar", %{conn: conn} do
    {:ok, view, _html} = live(conn, ~p"/works")
    
    view
    |> element("[data-testid='work-card-1']")
    |> render_click()
    
    assert has_element?(view, "[data-testid='work-sidebar']")
  end
end
```

```bash
# Run and iterate until passing:
mix test test/venus/works_test.exs
mix test test/venus_web/live/works_live/index_test.exs
# If failing: Read error, understand root cause, fix code, re-run
```

### Level 3: Integration Test
```bash
# Start the Phoenix server
mix phx.server

# Test in browser:
# 1. Navigate to http://localhost:4000/works
# 2. Verify works grid loads with infinite scroll
# 3. Click a work card to see sidebar populate
# 4. Test responsive design on mobile view

# Expected: Functional works page with sidebar interaction
```

## Final Validation Checklist
- [ ] All tests pass: `mix test`
- [ ] No compilation errors: `mix compile`
- [ ] No style issues: `mix credo --strict`
- [ ] Works page loads with grid layout
- [ ] Infinite scroll adds more works
- [ ] Work selection updates sidebar
- [ ] Actress profile displays correctly
- [ ] Photo gallery shows work pictures
- [ ] Mobile responsive sidebar works
- [ ] Navigation integration functional

---

## Anti-Patterns to Avoid
- ❌ Don't create new pagination patterns - use existing Repo.paginate
- ❌ Don't skip preloading associations for sidebar - causes N+1 queries
- ❌ Don't break existing WorkCard functionality
- ❌ Don't use different styling patterns - maintain Netflix theme
- ❌ Don't forget mobile responsive sidebar overlay
- ❌ Don't implement rating system without proper data structure

## Confidence Score: 8/10

**Reasoning:**
- **Strengths**: Strong existing patterns, clear implementation path, comprehensive context
- **Risks**: Two-column state management complexity, new component integration
- **Mitigation**: Detailed pseudocode, existing pattern references, progressive validation

This PRP provides comprehensive context for one-pass implementation success through proven patterns and detailed guidance.