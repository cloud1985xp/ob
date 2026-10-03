# Actresses Page Redesign - Modern UI/UX with Infinite Scroll

## Goal
Redesign the `/actresses` page from a basic table layout to a modern, Netflix-like interface with infinite scroll pagination, sidebar navigation, and responsive design. Transform the user experience from functional to engaging while maintaining performance.

## Why
- **User Experience**: Current table layout is outdated and doesn't showcase actresses effectively
- **Performance**: Infinite scroll with lazy loading improves page load times and user engagement
- **Visual Appeal**: Modern grid layout with rich media showcases content better
- **Mobile Experience**: Responsive design essential for mobile users
- **Consistency**: Aligns with existing homepage Netflix-like design patterns

## What
Convert `/actresses` route from Phoenix controller to LiveView with:

### Core Features
1. **Sidebar**: Scrollable list of hottest actresses with avatars, names, and ranks
2. **Main Grid**: Responsive grid layout displaying all actresses (30 per load)
3. **Infinite Scroll**: Automatic loading of next 30 actresses on scroll
4. **Search Functionality**: Real-time search by actress name
5. **Rich Cards**: Each actress card includes avatar, name, bio, rank, metrics, and profile link
6. **Responsive Design**: Mobile-first approach with Tailwind CSS

### Success Criteria
- [ ] Sidebar shows top 20 hottest actresses with lazy-loaded avatars
- [ ] Main grid loads 30 actresses initially, ordered by rank/popularity
- [ ] Infinite scroll triggers automatically when user reaches bottom
- [ ] Search filters actresses in real-time with debounced input
- [ ] All actress cards include required information and profile links
- [ ] Fully responsive on mobile, tablet, and desktop
- [ ] Performance optimized with lazy loading and efficient rendering

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://www.yellowduck.be/posts/scroll-events-and-infinite-pagination-in-phoenix-liveview
  why: Core infinite scroll implementation with phx-viewport-bottom events

- url: https://fly.io/phoenix-files/infinitely-scroll-images-in-liveview/
  why: LiveView streams pattern for infinite scroll with JavaScript hooks

- file: lib/venux_web/live/home_live/index.ex
  why: Existing LiveView pattern to follow for structure and event handling

- file: lib/venux_web/components/actress_card.ex
  why: Existing actress card component to adapt for grid layout

- file: lib/venux_web/ui.ex
  why: DaisyUI component patterns and lazy_img usage examples

- file: lib/venux/actresses.ex
  why: Existing context functions for actress data retrieval

- doc: https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#stream/3
  section: Stream functions for efficient list rendering
  critical: Stream management prevents memory issues with large lists
```

### Current Codebase Structure
```bash
lib/venux_web/
├── controllers/
│   ├── actress_controller.ex          # Current CRUD controller
│   └── actress_html/
│       └── index.html.heex           # Basic table layout to replace
├── live/
│   └── home_live/
│       ├── index.ex                  # LiveView pattern reference
│       └── index.html.heex           # UI patterns reference
├── components/
│   ├── actress_card.ex               # Existing card component
│   └── ui.ex                         # DaisyUI components + lazy_img
└── router.ex                         # Route configuration

lib/venux/
├── actresses.ex                      # Context module for data access
├── actress.ex                        # Actress schema
└── repo.ex                          # Scrivener pagination config
```

### Desired Codebase Structure with New Files
```bash
lib/venux_web/
├── live/
│   └── actresses_live/
│       ├── index.ex                  # NEW: Main LiveView module
│       └── index.html.heex           # NEW: Modern UI template
├── components/
│   ├── actress_grid_card.ex          # NEW: Optimized grid card component
│   └── actresses_sidebar.ex          # NEW: Sidebar component
└── router.ex                         # MODIFY: Update route to LiveView

lib/venux/
└── actresses.ex                      # MODIFY: Add search functionality

assets/js/
└── hooks/
    └── infinite_scroll.js            # NEW: JavaScript hook for scroll detection
```

### Known Gotchas & Library Quirks
```elixir
# CRITICAL: Scrivener is configured with page_size: 30 in Venus.Repo
# Use existing pagination but adapt for LiveView streams

# CRITICAL: lazy_img component requires data-src attribute
# Pattern: <.lazy_img src={Venus.Avatar.url({@actress.avatar, @actress}, :thumb)} />

# CRITICAL: DaisyUI components pattern from ui.ex
# Always use <.component_name> syntax, not raw HTML classes

# CRITICAL: LiveView streams require proper cleanup
# Use stream(:actresses, actress, at: -1) for append operations

# CRITICAL: Search + infinite scroll interaction
# Reset stream when search query changes, don't mix filtered and unfiltered results

# CRITICAL: Responsive grid classes
# Use Tailwind grid: grid-cols-1 md:grid-cols-2 lg:grid-cols-3 xl:grid-cols-4
```

## Implementation Blueprint

### Data Models and Structure
Current actress model is well-defined with all required fields:
```elixir
# Venus.Actress schema fields:
# - avatar: Venus.Avatar.Type (Waffle upload)
# - display_name: string (formatted name like "三上悠亜")
# - name: string (romanized name like "Yua Mikami")  
# - description: string (bio/description)
# - rank: integer (popularity ranking)
# - data: string (additional metadata as JSON)
# - state: integer (active/inactive status)

# Additional computed fields needed:
# - follower_count: derived from relationships
# - works_count: count of associated works
# - popularity_score: calculated metric
```

### Implementation Tasks (Sequential Order)

```yaml
Task 1 - Extend Context Layer:
MODIFY lib/venux/actresses.ex:
  - ADD search functionality to list_actresses/1
  - MODIFY to accept search params: %{"search" => query}
  - IMPLEMENT case-insensitive search on name and display_name
  - PRESERVE existing rank ordering and pagination

Task 2 - Create LiveView Module:
CREATE lib/venux_web/live/actresses_live/index.ex:
  - MIRROR structure from lib/venux_web/live/home_live/index.ex
  - IMPLEMENT mount/3 with initial data loading
  - ADD handle_event for "load-more", "search", "reset-search"
  - USE Phoenix.LiveView.stream for actress list management

Task 3 - Create Actress Grid Card Component:  
CREATE lib/venux_web/components/actress_grid_card.ex:
  - ADAPT from lib/venux_web/components/actress_card.ex
  - MODIFY for compact grid layout (not full-width)
  - INCLUDE avatar, name, bio (truncated), rank badge, metrics
  - USE lazy_img component for all images

Task 4 - Create Sidebar Component:
CREATE lib/venux_web/components/actresses_sidebar.ex:
  - USE hottest_actresses/1 function from context
  - IMPLEMENT scrollable list with fixed height
  - INCLUDE avatar + name + rank for each actress
  - MIRROR styling patterns from existing components

Task 5 - Create LiveView Template:
CREATE lib/venux_web/live/actresses_live/index.html.heex:
  - IMPLEMENT responsive layout: sidebar + main content
  - ADD search input with phx-debounce="300"
  - USE phx-update="stream" for actress grid
  - ADD phx-viewport-bottom for infinite scroll trigger

Task 6 - Add JavaScript Hook (Optional):
CREATE assets/js/hooks/infinite_scroll.js:
  - IMPLEMENT IntersectionObserver for scroll detection
  - TRIGGER "load-more" event when reaching bottom
  - HANDLE loading states and prevent duplicate requests

Task 7 - Update Router:  
MODIFY lib/venux_web/router.ex:
  - CHANGE actresses route from resources to live route
  - PRESERVE other actress routes (show, edit, etc.)

Task 8 - Add Responsive Styles:
MODIFY assets/css/app.css if needed:
  - ADD any custom grid utilities
  - ENSURE proper mobile responsiveness
```

### Pseudocode for Key Components

```elixir
# Task 2: LiveView Module Structure
defmodule VenusWeb.ActressesLive.Index do
  use VenusWeb, :live_view
  
  def mount(_params, _session, socket) do
    # PATTERN: Load initial data similar to HomeLive
    actresses = Actresses.list_actresses(%{"page" => 1})
    hottest = Actresses.hottest_actresses(20)
    
    socket = 
      socket
      |> assign(page_title: "Browse Actresses")
      |> assign(current_page: 1, search_query: "", has_more: true)
      |> assign(hottest_actresses: hottest)
      |> stream(:actresses, actresses.entries)
    
    {:ok, socket}
  end
  
  def handle_event("search", %{"query" => query}, socket) do
    # CRITICAL: Reset stream on search, don't mix results
    actresses = Actresses.list_actresses(%{"search" => query, "page" => 1})
    
    socket = 
      socket
      |> assign(search_query: query, current_page: 1)
      |> stream(:actresses, actresses.entries, reset: true)
    
    {:noreply, socket}
  end
  
  def handle_event("load-more", _params, socket) do
    # PATTERN: Scrivener pagination with stream append
    next_page = socket.assigns.current_page + 1
    params = %{"page" => next_page, "search" => socket.assigns.search_query}
    actresses = Actresses.list_actresses(params)
    
    socket = 
      socket
      |> assign(current_page: next_page, has_more: length(actresses.entries) > 0)
      |> stream(:actresses, actresses.entries, at: -1)
    
    {:noreply, socket}
  end
end
```

```elixir
# Task 3: Grid Card Component Structure  
defmodule VenusWeb.ActressGridCard do
  use Phoenix.Component
  use VenusWeb, :verified_routes
  import VenusWeb.UI, only: [lazy_img: 1]
  
  def actress_grid_card(assigns) do
    ~H"""
    <div class="bg-base-200 rounded-lg shadow-lg hover:shadow-xl transition-all duration-300 hover:scale-105">
      <!-- PATTERN: Use lazy_img for all images -->
      <.lazy_img 
        src={Venus.Avatar.url({@actress.avatar, @actress}, :thumb)}
        alt={@actress.display_name}
        class="w-full h-48 object-cover rounded-t-lg"
      />
      
      <div class="p-4">
        <div class="flex items-center justify-between mb-2">
          <h3 class="font-bold text-lg line-clamp-1">{@actress.display_name}</h3>
          <.badge class="badge-primary">#{@actress.rank}</.badge>
        </div>
        
        <p class="text-sm text-base-content/70 line-clamp-2 mb-3">{@actress.description}</p>
        
        <div class="flex justify-between items-center text-xs text-base-content/60 mb-3">
          <span>👥 {get_follower_count(@actress)} followers</span>
          <span>🎬 {get_works_count(@actress)} works</span>
        </div>
        
        <.link 
          href={~p"/actresses/#{@actress.id}"} 
          class="btn btn-primary btn-sm w-full"
        >
          View Profile
        </.link>
      </div>
    </div>
    """
  end
end
```

### Integration Points
```yaml
ROUTER:
  - change: lib/venux_web/router.ex line ~22
  - from: resources "/actresses", ActressController  
  - to: live "/actresses", ActressesLive.Index, :index

CONTEXT:
  - extend: lib/venux/actresses.ex list_actresses/1
  - add: search parameter handling with Ecto.Query
  - pattern: |
      def list_actresses(params \\ []) do
        search_query = params["search"]
        
        Actress
        |> maybe_search(search_query)
        |> order_by([desc: :rank])
        |> Repo.paginate(params)
      end

CSS:
  - verify: Tailwind grid utilities available
  - add if needed: Custom utilities for actress grid layout
```

## Validation Loop

### Level 1: Syntax & Compilation
```bash
# Run these FIRST - fix any errors before proceeding
mix compile                           # Elixir compilation
mix credo                            # Static analysis (mentioned in CLAUDE.md)

# Expected: No compilation errors or warnings
# If errors: Read carefully and fix Elixir syntax, imports, or module issues
```

### Level 2: Functionality Testing
```bash  
# Start the Phoenix server
mix phx.server

# Manual test checklist:
# 1. Navigate to http://localhost:4000/actresses
# 2. Verify initial load shows 30 actresses in grid
# 3. Check sidebar shows hottest actresses
# 4. Test search functionality with actress names
# 5. Scroll to bottom and verify infinite scroll loads more
# 6. Test responsive behavior on mobile view
# 7. Verify all links work correctly

# Expected: All functionality works smoothly without errors
# Check browser console for JavaScript errors
# Check server logs for Elixir errors
```

### Level 3: Performance & Edge Cases
```bash
# Test edge cases:
curl -X GET "http://localhost:4000/actresses" # Basic page load
# Test with search queries
# Test with no results scenarios
# Test infinite scroll until end of data

# Performance checks:
# - Verify lazy loading of images works
# - Check network tab for efficient data loading  
# - Ensure smooth scrolling performance
# - Test with slower network conditions
```

## Final Validation Checklist
- [ ] Code compiles without errors: `mix compile`
- [ ] No Credo warnings: `mix credo`
- [ ] All manual tests pass: Browse, search, infinite scroll
- [ ] Responsive design works on mobile
- [ ] Images lazy load correctly
- [ ] Search debouncing works properly
- [ ] Infinite scroll handles end-of-data gracefully
- [ ] Loading states provide good user feedback
- [ ] No console errors in browser
- [ ] Performance is smooth with large datasets

---

## Anti-Patterns to Avoid
- ❌ Don't create new DaisyUI patterns when existing ones work
- ❌ Don't skip lazy loading - performance is critical
- ❌ Don't mix search results with non-search results in stream
- ❌ Don't ignore mobile responsiveness
- ❌ Don't forget to handle empty states and loading states
- ❌ Don't use sync database calls in LiveView event handlers
- ❌ Don't hardcode pagination size - use Scrivener config

**PRP Confidence Score: 9/10** - Comprehensive context, clear implementation path, established patterns, proper validation gates.