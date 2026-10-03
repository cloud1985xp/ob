# ActressWorkset Homepage Feature Implementation

## Goal
Replace the "Latest Works Gallery" section in the HomeLive index page with a new ActressWorkset component system that displays pending works grouped by actress, with actress profiles on the left and horizontally scrollable works on the right. Enable user actions (Collect, Ignore, Like) on individual works.

## Why
- **User Experience**: Provides a more organized view of pending works by actress, making it easier for users to discover and interact with content from their favorite performers
- **Content Management**: Introduces a workflow for users to manage their content consumption with pending/collected/watching/viewed/omitted statuses
- **Engagement**: Action buttons enable immediate user feedback and content curation

## What
A new component-based system where:
- Each ActressWorkset shows an actress profile (left) and their pending works (right) in horizontal scroll
- Works are filtered by status = :pending and grouped by actress
- Users can perform actions: "Collect", "Ignore", "Like" on individual works
- Design maintains consistency with existing homepage aesthetic
- Fully responsive for mobile and desktop

### Success Criteria
- [ ] ActressWorkset components replace Latest Works Gallery
- [ ] Pending works are fetched and grouped by actress correctly
- [ ] Horizontal scrolling works on all screen sizes
- [ ] Action buttons function and update work status
- [ ] Existing homepage functionality remains intact
- [ ] Design is consistent with current aesthetic
- [ ] Mobile responsive layout works properly

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://hexdocs.pm/ecto/Ecto.Enum.html
  why: Ecto.Enum pattern for ActressWork status field conversion between integers and atoms

- file: lib/venux_web/live/home_live/index.ex
  why: Current HomeLive implementation, mount/3 function, event handlers pattern

- file: lib/venux_web/live/home_live/index.html.heex
  why: Current template structure, WorksGallery usage pattern to replace

- file: lib/venux_web/components/actress_profile.ex
  why: Existing ActressProfile component to reuse in ActressWorkset

- file: lib/venux_web/components/work_card.ex
  why: Existing WorkCard component to modify for action buttons

- file: lib/venux_web/components/works_gallery.ex
  why: Horizontal scrolling pattern and responsive design approach

- file: lib/venux/actresses/actress_work.ex
  why: Current ActressWork schema with state field

- file: lib/venux/works.ex
  why: Current Works context to extend with grouped query

- file: lib/venux/actresses.ex
  why: Actresses context for rank-based ordering
```

### Current Codebase Tree (relevant sections)
```bash
lib/
├── venux/
│   ├── actresses/
│   │   ├── actress.ex                 # Actress schema
│   │   └── actress_work.ex           # ActressWork schema (has state field)
│   ├── actresses.ex                  # Actresses context
│   └── works.ex                      # Works context (needs extension)
├── venux_web/
│   ├── components/
│   │   ├── actress_profile.ex        # Reusable actress profile component
│   │   ├── work_card.ex             # Work card component (modify for actions)
│   │   └── works_gallery.ex         # Horizontal scroll pattern reference
│   └── live/
│       └── home_live/
│           ├── index.ex             # HomeLive controller (modify mount + events)
│           └── index.html.heex      # Template (replace WorksGallery section)
```

### Desired Codebase Tree with New Files
```bash
lib/
├── venux_web/
│   └── components/
│       └── actress_workset.ex       # NEW: ActressWorkset component (profile + works)
# All other files: MODIFY existing files, no new files in contexts
```

### Known Gotchas & Library Quirks
```elixir
# CRITICAL: Ecto.Enum requires explicit integer mappings
field :state, Ecto.Enum, values: [pending: 0, collected: 1, watching: 2, viewed: 3, omitted: 4]

# GOTCHA: Complex grouping queries can cause N+1 problems
# PATTERN: Use preload and proper joins to avoid N+1 queries

# CRITICAL: Horizontal scroll needs proper CSS for mobile
# PATTERN: Use "overflow-x-auto scrollbar-hide" with proper touch handling

# GOTCHA: Action buttons in WorkCard need event.stopPropagation()
# PATTERN: prevent navigation when clicking action buttons inside links

# CRITICAL: LiveView event handlers need proper parameter passing
# PATTERN: phx-click with phx-value-work_id and phx-value-action
```

## Implementation Blueprint

### Data Models and Structure

Modify existing ActressWork schema to use Ecto.Enum for status management:
```elixir
# In Venus.Actresses.ActressWork
field :state, Ecto.Enum, values: [pending: 0, collected: 1, watching: 2, viewed: 3, omitted: 4]
```

Add query function for grouped pending works:
```elixir
# In Venus.Works context
def get_pending_works_by_actress(limit_per_actress \\ 5) do
  # Returns list of %{actress: %Actress{}, works: [%ActressWork{}, ...]}
end
```

### List of Tasks (Implementation Order)

```yaml
Task 1 - Add Ecto.Enum to ActressWork:
MODIFY lib/venux/actresses/actress_work.ex:
  - FIND: field :state, :integer
  - REPLACE with: field :state, Ecto.Enum, values: [pending: 0, collected: 1, watching: 2, viewed: 3, omitted: 4]
  - UPDATE changeset/2 to handle enum validation

Task 2 - Create Pending Works Query:
MODIFY lib/venux/works.ex:
  - ADD function: get_pending_works_by_actress/1
  - PATTERN: Complex query with joins, grouping, and preloading
  - ORDER BY: actress.rank ASC, work.published_on DESC
  - LIMIT: works per actress (default 5)

Task 3 - Create ActressWorkset Component:
CREATE lib/venux_web/components/actress_workset.ex:
  - COMPONENT: actress_workset/1 with actress and works attributes
  - LAYOUT: CSS Grid with actress profile left, works right
  - RESPONSIVE: Stack vertically on mobile
  - REUSE: ActressProfile and WorkCard components

Task 4 - Modify WorkCard for Actions:
MODIFY lib/venux_web/components/work_card.ex:
  - ADD attr: actions (list of action maps)
  - MODIFY: work_card_content to render dynamic action buttons
  - PATTERN: Use phx-click with action type and work_id
  - PRESERVE: existing functionality and disable_navigation logic

Task 5 - Update HomeLive Data Loading:
MODIFY lib/venux_web/live/home_live/index.ex:
  - MODIFY load_homepage_data/1: replace latest_works assignment
  - ADD: pending_works_by_actress assignment using new query
  - PRESERVE: all existing assignments and functionality

Task 6 - Add Action Event Handlers:
MODIFY lib/venux_web/live/home_live/index.ex:
  - ADD handle_event/3 for "collect_work", "ignore_work", "like_work"
  - PATTERN: Update work status and reload data
  - PRESERVE: all existing event handlers

Task 7 - Update Homepage Template:
MODIFY lib/venux_web/live/home_live/index.html.heex:
  - FIND: Latest Works Gallery section (lines ~24-32)
  - REPLACE with: ActressWorkset loop rendering
  - PRESERVE: Hottest Works Gallery and Photo Gallery sections
  - MAINTAIN: existing styling and layout structure
```

### Task-Specific Pseudocode

```elixir
# Task 2: Pending Works Query
def get_pending_works_by_actress(limit_per_actress \\ 5) do
  # PATTERN: Use window functions for ranking within groups
  query = from w in ActressWork,
    join: a in Actress, on: w.actress_id == a.id,
    where: w.state == :pending,
    windows: [
      actress_window: [partition_by: w.actress_id, order_by: [desc: w.published_on]]
    ],
    select: %{
      actress_id: w.actress_id,
      work: w,
      actress: a,
      row_number: row_number() |> over(:actress_window)
    },
    where: row_number() |> over(:actress_window) <= ^limit_per_actress,
    order_by: [asc: a.rank, desc: w.published_on],
    preload: [actress: a]
  
  # CRITICAL: Group results by actress to avoid N+1
  results = Repo.all(query)
  Enum.group_by(results, & &1.actress, & &1.work)
end

# Task 3: ActressWorkset Component Structure
def actress_workset(assigns) do
  ~H"""
  <div class="grid grid-cols-1 lg:grid-cols-[300px_1fr] gap-6 bg-white border border-gray-100 rounded-2xl p-6">
    <!-- Actress Profile (Left/Top) -->
    <div class="flex-shrink-0">
      <ActressProfile.actress_profile actress={@actress} class="h-fit" />
    </div>
    
    <!-- Works Horizontal Scroll (Right/Bottom) -->
    <div class="overflow-x-auto">
      <div class="flex space-x-4 pb-4">
        <%= for work <- @works do %>
          <div class="flex-none w-64">
            <WorkCard.work_card 
              work={work} 
              actions={[
                %{type: "collect", label: "Collect", icon: "plus"},
                %{type: "ignore", label: "Ignore", icon: "x"},
                %{type: "like", label: "Like", icon: "heart"}
              ]}
              disable_navigation={false}
            />
          </div>
        <% end %>
      </div>
    </div>
  </div>
  """
end
```

### Integration Points
```yaml
DATABASE:
  - enum: ActressWork.state field uses Ecto.Enum
  - query: Complex grouping query with window functions for performance

COMPONENTS:
  - reuse: ActressProfile component unchanged
  - modify: WorkCard to accept actions parameter
  - create: ActressWorkset for layout coordination

LIVEVIEW:
  - mount: Load pending works grouped by actress
  - events: Handle collect/ignore/like actions with status updates
  - template: Replace Latest Works section with ActressWorkset loop

STYLING:
  - pattern: Follow existing Tailwind classes and color scheme
  - responsive: CSS Grid with responsive breakpoints
  - scroll: Horizontal overflow with proper mobile touch handling
```

## Validation Loop

### Level 1: Compilation & Syntax
```bash
# Run these FIRST - fix any errors before proceeding
mix compile                          # Check Elixir compilation
mix credo --strict                  # Code quality and style
# Expected: No compilation errors or credo issues
```

### Level 2: Unit Tests
```elixir
# CREATE test/venux/works_test.exs additions:
describe "get_pending_works_by_actress/1" do
  test "returns works grouped by actress ordered by rank" do
    # Create test data with multiple actresses and pending works
    result = Works.get_pending_works_by_actress()
    
    assert length(result) > 0
    # Verify grouping and ordering
  end
  
  test "limits works per actress" do
    # Test limit parameter works correctly
  end
  
  test "returns empty list when no pending works" do
    # Edge case handling
  end
end

# CREATE test/venux_web/components/actress_workset_test.exs:
describe "actress_workset/1" do
  test "renders actress profile and works" do
    # Test component rendering with mock data
  end
end
```

```bash
# Run and iterate until passing:
mix test test/venux/works_test.exs
mix test test/venux_web/components/actress_workset_test.exs
# If failing: Read error, fix code, re-run tests
```

### Level 3: Integration Test
```bash
# Start Phoenix server
mix phx.server

# Manual testing checklist:
# 1. Visit http://localhost:4000 
# 2. Verify ActressWorkset components appear instead of Latest Works
# 3. Test horizontal scrolling on works
# 4. Click action buttons (Collect, Ignore, Like)
# 5. Test responsive layout on mobile size
# 6. Verify existing features still work (chat, navigation)

# Expected: Homepage loads with new ActressWorkset layout, all actions functional
```

## Final Validation Checklist
- [ ] All tests pass: `mix test`
- [ ] No compilation errors: `mix compile`
- [ ] Code quality passes: `mix credo`
- [ ] Homepage displays ActressWorkset components
- [ ] Action buttons work (collect/ignore/like)
- [ ] Horizontal scrolling functions properly
- [ ] Mobile responsive layout works
- [ ] Existing homepage features unchanged
- [ ] No console errors in browser
- [ ] Performance acceptable (no N+1 queries)

## Anti-Patterns to Avoid
- ❌ Don't create N+1 queries in the grouping function
- ❌ Don't break existing WorkCard functionality when adding actions
- ❌ Don't ignore mobile responsiveness in horizontal scroll
- ❌ Don't remove existing event handlers in HomeLive.Index
- ❌ Don't hardcode actress limits or work limits
- ❌ Don't skip preloading associations in complex queries
- ❌ Don't use inline styles instead of Tailwind classes

---

## Confidence Score: 8/10

This PRP provides comprehensive context for implementing the ActressWorkset feature with:
- ✅ Complete codebase analysis and existing patterns identified
- ✅ Detailed implementation plan with specific file changes
- ✅ Ecto.Enum best practices and gotchas documented
- ✅ Component reuse strategy clearly defined
- ✅ Responsive design considerations included
- ✅ Testing strategy with specific test cases
- ✅ Edge cases and error handling planned

The main complexity lies in the database grouping query and ensuring proper event handling, but with the documented patterns and validation steps, this should be achievable in a single implementation pass.