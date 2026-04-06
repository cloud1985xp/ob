name: "Works Detail Page Implementation - Context-Rich PRP"
description: |

## Purpose
Implement a comprehensive work detail page at `/works/:id` that displays work information, actress profile, media player, comments, tags, and photo gallery with real-time interactions.

## Core Principles
1. **Context is King**: Leveraging existing codebase patterns and components
2. **Validation Loops**: Executable tests and checks for iterative refinement  
3. **Information Dense**: Following established Venus codebase conventions
4. **Progressive Success**: Build core functionality first, then enhance
5. **Global rules**: Follow all rules in CLAUDE.md

---

## Goal
Create a modern, responsive work detail page (`/works/:id`) that displays comprehensive work information including media player, actress details, comments system, tags, and photo gallery - similar to Netflix/YouTube content pages but tailored for the Venus platform.

## Why
- **User Experience**: Provide engaging detail view for works to increase user engagement
- **Content Discovery**: Enable users to explore work details, related actress information, and associated media
- **Social Interaction**: Allow commenting and tagging to build community around content
- **Mobile-First**: Ensure responsive design for mobile and desktop users

## What
A LiveView-powered page that displays:
- Work details (title, description, release date, cover, state badge)
- Actress information card (reusable component)
- Media player for videos/photos with custom controls
- Real-time comments system with form for new comments
- Tag display and selection functionality  
- Responsive photo grid using existing PhotoCard component

### Success Criteria
- [ ] Navigate to `/works/:id` renders work details without errors
- [ ] Media player supports both video and photo content with controls
- [ ] Actress card displays correctly with proper styling
- [ ] Comments display and new comment form works
- [ ] Tags are displayed and selectable
- [ ] Photo grid is responsive and uses PhotoCard component
- [ ] Page is mobile-friendly and follows existing design patterns
- [ ] All interactions work without JavaScript errors

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- url: https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html
  why: Core LiveView patterns, mount/3, handle_params/3, handle_event/3

- url: https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Elements/video
  why: HTML5 video element and controls API for media player

- url: https://freshman.tech/custom-html5-video/
  why: Custom video player implementation guide with JavaScript controls

- url: https://dockyard.com/blog/2019/11/25/phoenix-liveview-comment-and-reply
  why: Phoenix LiveView comment system implementation patterns

- file: lib/venux_web/live/actresses_live/show.ex
  why: Existing LiveView pattern to follow for mount, handle_params, event handling

- file: lib/venux_web/components/work_card.ex  
  why: Work data structure, state badges, styling patterns to reuse

- file: lib/venux_web/components/actress_profile.ex
  why: Base pattern for creating ActressCard component

- file: lib/venux_web/components/photo_card.ex
  why: Photo grid implementation and styling patterns

- file: lib/venux/works.ex
  why: get_work_with_details/1 function and Works context patterns

- file: lib/venux/actresses/actress_work.ex
  why: ActressWork schema with state enum and associations
```

### Current Codebase Structure
```bash
lib/venux_web/
├── live/
│   ├── works_live/
│   │   ├── index.ex           # EXISTS - patterns to follow
│   │   └── index.html.heex    # EXISTS
│   │   # show.ex - MISSING (to be created)
│   │   # show.html.heex - MISSING (to be created)
│   └── shared_data.ex         # SharedData mount hook
├── components/
│   ├── work_card.ex           # Work display patterns
│   ├── actress_profile.ex     # Actress display patterns  
│   ├── photo_card.ex          # Photo grid patterns
│   └── core_components.ex     # Base components
├── router.ex                  # Route exists: live "/works/:id", WorksLive.Show
└── ...

lib/venux/
├── works.ex                   # get_work_with_details/1 function
└── actresses/
    └── actress_work.ex        # Schema with associations
```

### Desired Codebase Structure After Implementation
```bash
lib/venux_web/
├── live/
│   └── works_live/
│       ├── show.ex            # NEW - Main LiveView module
│       └── show.html.heex     # NEW - Template with sections
├── components/
│   ├── media_player.ex        # NEW - Video/photo player
│   ├── actress_card.ex        # NEW - Compact actress display
│   └── comments_list.ex       # NEW - Reusable comments component
└── ...

test/venux_web/live/works_live/
└── show_test.exs              # NEW - LiveView tests
```

### Known Gotchas & Library Quirks
```elixir
# CRITICAL: Work IDs can be string or integer - handle both types
work_id = String.to_integer(id) # May raise ArgumentError
work = Works.get_work_with_details(work_id) # Returns nil if not found

# CRITICAL: State enum values must match ActressWork schema
# Values: [:pending, :archived, :collected, :viewed, :omitted]
state_badge_colors = %{
  pending: "bg-red-500",
  archived: "bg-purple-500", 
  collected: "bg-green-500",
  viewed: "bg-blue-500",
  omitted: "bg-gray-500"
}

# CRITICAL: LiveView streams require unique IDs
stream(:comments, comments) # Each comment needs %{id: unique_id}

# CRITICAL: SharedData mount hook adds layout data
on_mount {VenusWeb.SharedData, :layout_data}

# CRITICAL: DaisyUI and Tailwind classes for responsive design
class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4"

# CRITICAL: Venus.Cover.url/2 for work cover images
Venus.Cover.url({work.cover, work}, :title)

# CRITICAL: Venus.Picture.url/2 for work pictures  
Venus.Picture.url({picture.file, picture}, :original)
```

## Implementation Blueprint

### Data Models and Structure
Work detail page relies on existing schemas:
```elixir
# ActressWork schema fields (lib/venux/actresses/actress_work.ex)
%ActressWork{
  id: integer,
  title: string,
  description: string, 
  published_on: date,
  cover: Venus.Cover.Type,
  state: enum [:pending, :archived, :collected, :viewed, :omitted],
  actress: %Actress{}, # via belongs_to association
  pictures: [%ActressWorkPicture{}] # via has_many association
}

# Mock data structures to implement:
mock_comments = [
  %{id: 1, text: "Amazing work!", author: "User123", timestamp: "2 hours ago"},
  %{id: 2, text: "Love the cinematography", author: "FilmFan", timestamp: "1 day ago"}
]

mock_tags = ["Drama", "Romance", "HD", "2024", "Featured"]
mock_popular_tags = ["Action", "Comedy", "Drama", "Romance", "Thriller"] 
```

### Ordered Task List

```yaml
Task 1 - Core LiveView Setup:
CREATE lib/venux_web/live/works_live/show.ex:
  - MIRROR pattern from: lib/venux_web/live/actresses_live/show.ex
  - IMPLEMENT mount/3 with basic assigns
  - IMPLEMENT handle_params/3 to load work with Works.get_work_with_details/1
  - ADD error handling for missing works (redirect or 404)
  - ADD SharedData mount hook: on_mount {VenusWeb.SharedData, :layout_data}

MODIFY lib/venux_web/live/shared_data.ex:
  - ADD pattern: get_current_path(VenusWeb.WorksLive.Show), do: "/works"

Task 2 - Basic Template Structure:
CREATE lib/venux_web/live/works_live/show.html.heex:
  - FOLLOW responsive layout patterns from existing templates
  - CREATE hero section with work details
  - CREATE sections: actress-card, media-player, comments, tags, photos-grid
  - USE Tailwind classes for responsive design

Task 3 - Work Information Display:
MODIFY show.html.heex:
  - DISPLAY work title, description, published_on (formatted)
  - DISPLAY cover image using Venus.Cover.url/2
  - ADD state badge MIRRORING work_card.ex patterns
  - ENSURE mobile-responsive layout

Task 4 - ActressCard Component:
CREATE lib/venux_web/components/actress_card.ex:
  - MIRROR pattern from: lib/venux_web/components/actress_profile.ex
  - MODIFY to be more compact for detail page usage
  - PRESERVE styling patterns and lazy image loading
  - ADD attr :actress, :map, required: true

Task 5 - MediaPlayer Component:
CREATE lib/venux_web/components/media_player.ex:
  - CREATE attrs: media_url, media_type, poster_url
  - IMPLEMENT HTML5 video element with custom controls
  - ADD controls: play/pause, volume, fullscreen
  - HANDLE image display mode for photo content
  - FOLLOW existing component styling patterns

Task 6 - Comments System:
CREATE lib/venux_web/components/comments_list.ex:
  - CREATE reusable component for comment display
  - IMPLEMENT stream pattern for real-time updates
  - ADD attrs: comments (list), show_form (boolean)

MODIFY show.ex:
  - ADD mock_comments to assigns in mount/3
  - IMPLEMENT handle_event("add_comment", params, socket)
  - ADD comment validation and feedback

MODIFY show.html.heex:
  - INTEGRATE comments_list component
  - ADD comment form with phx-submit="add_comment"

Task 7 - Tags System:
MODIFY show.ex:
  - ADD mock_tags and mock_popular_tags to assigns
  - IMPLEMENT handle_event("add_tag", params, socket)
  - IMPLEMENT handle_event("remove_tag", params, socket)

MODIFY show.html.heex:
  - DISPLAY current tags with remove buttons
  - ADD tag selection from popular tags
  - CREATE tag input field for custom tags

Task 8 - Photos Grid:
MODIFY show.ex:
  - LOAD work pictures via work.pictures association
  - STREAM pictures for performance

MODIFY show.html.heex:
  - CREATE responsive grid using PhotoCard component
  - MIRROR grid patterns from existing photo galleries
  - HANDLE empty state when no pictures exist

Task 9 - Tests and Validation:
CREATE test/venux_web/live/works_live/show_test.exs:
  - MIRROR test patterns from existing LiveView tests
  - TEST mount with valid work ID
  - TEST mount with invalid work ID (404 handling)
  - TEST comment addition functionality
  - TEST tag management functionality
```

### Per Task Pseudocode

```elixir
# Task 1 - Core LiveView Setup
defmodule VenusWeb.WorksLive.Show do
  use VenusWeb, :live_view
  alias Venus.Works
  
  on_mount {VenusWeb.SharedData, :layout_data}
  
  def mount(_params, _session, socket) do
    socket = assign(socket,
      page_title: "Work Details",
      work: nil,
      comments: [],
      tags: [],
      popular_tags: ["Action", "Comedy", "Drama"]
    )
    {:ok, socket}
  end
  
  def handle_params(%{"id" => id}, _uri, socket) do
    # CRITICAL: Handle both string and integer IDs
    work_id = String.to_integer(id)
    work = Works.get_work_with_details(work_id)
    
    # PATTERN: Handle missing work gracefully
    case work do
      nil -> 
        {:noreply, redirect(socket, to: ~p"/works")}
      work ->
        socket = assign(socket, 
          work: work,
          page_title: work.title,
          comments: mock_comments(),
          tags: mock_tags()
        )
        {:noreply, socket}
    end
  end
end

# Task 5 - MediaPlayer Component
defmodule VenusWeb.MediaPlayer do
  use Phoenix.Component
  
  attr :media_url, :string, required: true
  attr :media_type, :string, default: "video" # "video" or "image"
  attr :poster_url, :string, default: nil
  
  def media_player(assigns) do
    ~H"""
    <div class="relative w-full bg-black rounded-lg overflow-hidden">
      <%= if @media_type == "video" do %>
        <video 
          controls 
          class="w-full h-auto max-h-96"
          poster={@poster_url}
          preload="metadata"
        >
          <source src={@media_url} type="video/mp4">
          Your browser does not support video playback.
        </video>
      <% else %>
        <img 
          src={@media_url} 
          alt="Work media"
          class="w-full h-auto max-h-96 object-contain"
        >
      <% end %>
    </div>
    """
  end
end
```

### Integration Points

```yaml
ROUTES:
  - existing: live "/works/:id", WorksLive.Show, :show (in router.ex)
  - update: SharedData.get_current_path/1 pattern

COMPONENTS:
  - reuse: Venus.Cover.url/2 for work cover images
  - reuse: Venus.Picture.url/2 for work pictures
  - reuse: PhotoCard component for pictures grid
  - reuse: UI.lazy_img for image loading

DATABASE:
  - use existing: Works.get_work_with_details/1 function
  - associations: work.actress, work.pictures (already defined)

STYLING:
  - follow: Tailwind responsive patterns from existing components
  - use: DaisyUI components where applicable
  - match: Color schemes from work_card.ex and actress_profile.ex
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Run these FIRST - fix any errors before proceeding
mix compile                    # Check for compilation errors
mix format                     # Auto-format code
mix credo --strict            # Code quality checks

# Expected: No errors. If errors, READ the error and fix.
```

### Level 2: Unit Tests
```elixir
# CREATE test/venux_web/live/works_live/show_test.exs
defmodule VenusWeb.WorksLive.ShowTest do
  use VenusWeb.ConnCase
  import Phoenix.LiveViewTest
  
  test "displays work details", %{conn: conn} do
    # Test with existing work ID from your database
    {:ok, view, html} = live(conn, ~p"/works/1")
    
    assert html =~ "Work Title"  # Adjust based on actual data
    assert has_element?(view, "[data-role='work-title']")
  end
  
  test "handles missing work", %{conn: conn} do
    # Test with non-existent work ID
    assert {:error, {:redirect, %{to: "/works"}}} = 
           live(conn, ~p"/works/99999")
  end
  
  test "adds comment", %{conn: conn} do
    {:ok, view, _html} = live(conn, ~p"/works/1")
    
    view
    |> form("[data-role='comment-form']", comment: %{text: "Great work!"})
    |> render_submit()
    
    assert has_element?(view, "[data-role='comment']", "Great work!")
  end
end
```

```bash
# Run and iterate until passing:
mix test test/venux_web/live/works_live/show_test.exs
# If failing: Read error, understand root cause, fix code, re-run
```

### Level 3: Integration Test
```bash
# Start the Phoenix server
mix phx.server

# Test the page manually in browser:
# 1. Navigate to http://localhost:4000/works/1
# 2. Verify work details display correctly
# 3. Test media player controls
# 4. Add a comment and verify it appears
# 5. Try adding/removing tags
# 6. Check mobile responsiveness

# Expected: Page loads without errors, all interactions work
# If error: Check logs for stack trace and fix issues
```

## Final Validation Checklist
- [ ] All tests pass: `mix test`
- [ ] No compilation errors: `mix compile`
- [ ] Code formatted: `mix format --check-formatted`
- [ ] No credo warnings: `mix credo --strict`
- [ ] Manual test successful: Navigate to `/works/1` and test all features
- [ ] Error cases handled: Missing work IDs redirect properly
- [ ] Mobile responsive: Test on different screen sizes
- [ ] Components reusable: ActressCard and CommentsList can be used elsewhere

---

## Anti-Patterns to Avoid
- ❌ Don't create new LiveView patterns when existing ones work (follow ActressesLive.Show)
- ❌ Don't skip error handling for missing works - handle gracefully
- ❌ Don't ignore responsive design - use existing Tailwind patterns
- ❌ Don't reinvent component styling - reuse existing patterns from WorkCard/ActressProfile
- ❌ Don't hardcode mock data directly in templates - use assigns
- ❌ Don't forget to handle both string and integer work IDs
- ❌ Don't create components without proper attrs validation
- ❌ Don't skip the SharedData mount hook - needed for layout consistency

## Confidence Level: 8/10

**Reasoning**: Comprehensive context provided including existing patterns, external documentation, clear task ordering, validation gates, and error handling strategy. The codebase has strong existing patterns to follow, and the external research provides solid technical foundation. Main complexity is in integration and styling consistency, but detailed context should enable successful one-pass implementation.