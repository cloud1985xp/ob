# PRP: Works Show Page Modern Redesign

## Goal
Redesign the WorksLive.Show page at `/works/:id` to create a modern, immersive viewing experience inspired by Netflix, Plex, and Disney+ streaming platforms. The new design should feature a cinematic hero section with background carousel, improved content organization, and enhanced user engagement features.

## Why
- **Enhanced User Experience**: Create a more engaging and visually appealing work details page
- **Modern Design Language**: Align with contemporary streaming platform aesthetics users expect
- **Improved Information Architecture**: Better organization of work details, actress info, and user interactions
- **Mobile Responsiveness**: Ensure excellent experience across all device sizes
- **Increased User Engagement**: Add interactive elements (like, favorite, watch now) to drive user actions

## What
Transform the current 3-column layout into a modern design featuring:

1. **Hero Section** (almost full viewport height):
   - Black background with carousel of work images
   - Fade transitions between background images
   - Images positioned top-right with gradient overlay effect
   - Work information prominently displayed on the left
   - Action buttons for user interactions

2. **Content Section** (3-column layout):
   - Actress profile (reuse existing component)
   - Photo gallery with related works from same actress
   - Comments section (reuse existing component)

3. **Responsive Design**: Optimized for desktop, tablet, and mobile devices

### Success Criteria
- [ ] Hero section with full-height carousel background
- [ ] Smooth fade transitions between background images
- [ ] Work info displayed with modern typography and spacing
- [ ] Functional like, favorite, and watch now buttons
- [ ] Related works display (5 random works from same actress)
- [ ] Responsive layout works on mobile, tablet, and desktop
- [ ] Loading performance is acceptable (<3s initial load)
- [ ] Accessibility standards maintained (keyboard navigation, screen readers)

## All Needed Context

### Documentation & References
```yaml
# Phoenix LiveView Patterns
- file: lib/venux_web/live/works_live/show.ex
  why: Current implementation patterns, event handlers, data loading

- file: lib/venux_web/live/works_live/show.html.heex  
  why: Current layout structure and Tailwind CSS patterns

# Existing Components to Reuse
- file: lib/venux_web/components/actress_profile.ex
  why: Reuse for column 1 - already has modern styling and interactions

- file: lib/venux_web/components/work_card.ex
  why: Reuse for related works display with existing hover effects

- file: lib/venux_web/components/photo_gallery.ex
  why: Reuse photo_grid function for work pictures gallery

- file: lib/venux_web/components/comments_list.ex
  why: Reuse for column 3 - already has form handling and styling

# Data Layer
- file: lib/venux/works.ex
  why: Current data fetching patterns, need to add related works function

- file: lib/venux/actresses/actress_work.ex
  why: Understanding work schema and relationships

# UI Patterns and Styling
- url: https://www.figma.com/community/file/1348399379890177021/netflix-ui-2024
  why: Netflix UI patterns for hero sections and content layout

- url: https://www.plex.tv/blog/choose-your-own-adventure-introducing-modern-layout/
  why: Plex modern layout principles for tablet/desktop optimization

- url: https://tailwindcss.com/docs/background-image
  why: CSS gradient and background image techniques for carousel
```

### Current Codebase Structure
```bash
lib/venux_web/
├── live/works_live/
│   ├── show.ex                 # Current LiveView implementation
│   └── show.html.heex          # Current template (to be redesigned)
├── components/
│   ├── actress_profile.ex      # Reuse in column 1
│   ├── work_card.ex           # Reuse for related works
│   ├── photo_gallery.ex       # Reuse for photo grid
│   └── comments_list.ex       # Reuse in column 3
└── router.ex                  # Route: live "/works/:id", WorksLive.Show, :show

lib/venux/
├── works.ex                   # Data layer functions
└── actresses/
    └── actress_work.ex        # Work schema with pictures relationship
```

### Desired Codebase Structure (Files to Add)
```bash
lib/venux_web/components/
├── background_carousel.ex      # Hero background carousel with fade effects
├── work_hero_section.ex       # Complete hero section layout component  
├── work_action_buttons.ex     # Like, favorite, watch now buttons
└── related_works.ex           # Shows 5 works from same actress

lib/venux/
└── works.ex                   # Add get_related_works_by_actress/3 function
```

### Known Gotchas & Library Quirks
```elixir
# CRITICAL: Phoenix LiveView patterns
# Always preload associations in queries to avoid N+1 problems
work = Repo.one(from w in ActressWork, preload: [:actress, :pictures])

# CRITICAL: Tailwind CSS responsive design
# Use mobile-first approach: base styles, then sm:, md:, lg:, xl: modifiers
# Example: "w-full lg:w-1/2" (full width mobile, half width large screens)

# GOTCHA: CSS animations and LiveView
# Use CSS transitions/animations instead of JavaScript for better performance
# Alpine.js can be used for simple interactions without full JS

# GOTCHA: Image loading optimization
# Use Venus.Cover.url({work.cover, work}, :title) pattern for optimized images
# Always provide alt text and loading="lazy" for performance

# CRITICAL: Stream updates for pictures
# Use phx-update="stream" for photo galleries to handle dynamic updates
# Pattern: |> stream(:pictures, work.pictures || [])
```

## Implementation Blueprint

### Data Models and Backend Functions
Add to Venus.Works module:
```elixir
def get_related_works_by_actress(actress_id, current_work_id, limit \\ 5) do
  from(w in ActressWork,
    where: w.actress_id == ^actress_id and w.id != ^current_work_id and w.state == :collected,
    order_by: fragment("RANDOM()"),
    limit: ^limit,
    preload: :actress
  )
  |> Repo.all()
end

def get_carousel_images(work) do
  work.pictures
  |> Enum.filter(&(&1.width > 300))
  |> Enum.take(5)  # Limit for performance
end
```

### Task Implementation Order

```yaml
Task 1: Backend Data Functions
MODIFY lib/venux/works.ex:
  - ADD get_related_works_by_actress/3 function
  - ADD get_carousel_images/1 helper function
  - PATTERN: Follow existing query patterns with preloading

Task 2: Create Background Carousel Component
CREATE lib/venux_web/components/background_carousel.ex:
  - PATTERN: Phoenix Component with CSS animations
  - HANDLE: Image cycling, fade transitions, gradient overlay
  - RESPONSIVE: Scale properly on mobile devices

Task 3: Create Action Buttons Component  
CREATE lib/venux_web/components/work_action_buttons.ex:
  - PATTERN: Follow existing button patterns from work_card.ex
  - EVENTS: phx-click handlers for like, favorite, watch_now
  - STYLING: Modern button design with hover effects

Task 4: Create Work Hero Section Component
CREATE lib/venux_web/components/work_hero_section.ex:
  - COMBINE: BackgroundCarousel + work info + ActionButtons
  - LAYOUT: CSS Grid/Flexbox for positioning
  - RESPONSIVE: Stack vertically on mobile

Task 5: Create Related Works Component
CREATE lib/venux_web/components/related_works.ex:
  - REUSE: VenusWeb.WorkCard.work_card component
  - PATTERN: Horizontal scrolling grid layout
  - LIMIT: 5 works maximum for performance

Task 6: Update LiveView Module
MODIFY lib/venux_web/live/works_live/show.ex:
  - ADD assigns: related_works, carousel_images, action_states
  - ADD event handlers: like_work, favorite_work, watch_now
  - MODIFY handle_params/3 to fetch related works
  - KEEP existing comment and tag handlers

Task 7: Redesign Template Layout
MODIFY lib/venux_web/live/works_live/show.html.heex:
  - REPLACE current layout with hero section
  - RESTRUCTURE 3-column section below hero
  - MAINTAIN responsive mobile header
  - PRESERVE existing data attributes for testing

Task 8: Styling and Polish
REFINE all components:
  - GRADIENT overlays and visual effects
  - SMOOTH animations and transitions  
  - RESPONSIVE breakpoints and mobile optimization
  - ACCESSIBILITY improvements (focus states, ARIA labels)
```

### Component Pseudocode

```elixir
# BackgroundCarousel Component
defmodule VenusWeb.BackgroundCarousel do
  attr :images, :list, required: true
  attr :class, :string, default: ""
  
  def background_carousel(assigns) do
    ~H"""
    <div class={["relative w-full h-full overflow-hidden", @class]} 
         phx-hook="BackgroundCarousel" id="bg-carousel">
      <!-- Carousel Images -->
      <%= for {image, index} <- Enum.with_index(@images) do %>
        <div class={["absolute inset-0 transition-opacity duration-1000", 
                     if(index == 0, do: "opacity-100", else: "opacity-0")]}
             data-slide={index}>
          <img src={image.src} alt="" class="w-full h-full object-cover" />
        </div>
      <% end %>
      
      <!-- Gradient Overlay -->
      <div class="absolute inset-0 bg-gradient-to-bl from-transparent via-black/30 to-black/80"></div>
    </div>
    """
  end
end

# WorkHeroSection Component  
defmodule VenusWeb.WorkHeroSection do
  def work_hero_section(assigns) do
    ~H"""
    <section class="relative min-h-screen bg-black overflow-hidden">
      <!-- Background Carousel (right side) -->
      <div class="absolute top-0 right-0 w-full lg:w-3/5 h-full">
        <.background_carousel images={@carousel_images} />
      </div>
      
      <!-- Work Info (left side) -->
      <div class="relative z-10 p-6 lg:p-12 w-full lg:w-2/5 min-h-screen flex flex-col justify-center">
        <!-- Work Cover, Title, Description, etc. -->
        <!-- Action Buttons -->
      </div>
    </section>
    """
  end
end
```

### Integration Points
```yaml
LIVEVIEW ASSIGNS:
  - related_works: List of 5 random works from same actress  
  - carousel_images: Filtered pictures for background carousel
  - like_status: Boolean for like button state
  - favorite_status: Boolean for favorite button state

ROUTES (existing):
  - live "/works/:id", WorksLive.Show, :show
  - No new routes needed

CSS/JAVASCRIPT HOOKS:
  - BackgroundCarousel: Auto-advance slides every 5 seconds
  - Intersection Observer: Pause carousel when not visible
  - Smooth scrolling: Between sections

EVENT HANDLERS:
  - like_work: Toggle like state, show feedback
  - favorite_work: Toggle favorite state, show feedback  
  - watch_now: Navigate to video player or show modal
```

## Validation Loop

### Level 1: Syntax & Style
```bash
# Elixir formatting and compilation
mix format
mix compile

# Code quality analysis
mix credo --strict

# Expected: No warnings or errors. Clean compilation.
```

### Level 2: Component Unit Tests
```elixir
# test/venux_web/components/background_carousel_test.exs
defmodule VenuxWeb.BackgroundCarouselTest do
  use VenuxWeb.ConnCase, async: true
  import Phoenix.LiveViewTest

  test "renders carousel with images" do
    images = [%{src: "/image1.jpg"}, %{src: "/image2.jpg"}]
    
    html = 
      render_component(&VenuxWeb.BackgroundCarousel.background_carousel/1, 
                      images: images)
    
    assert html =~ "bg-carousel"
    assert html =~ "image1.jpg"
    assert html =~ "image2.jpg"
  end
end

# test/venux_web/live/works_live/show_test.exs  
test "handles like_work event", %{conn: conn} do
  work = insert(:actress_work)
  
  {:ok, view, _html} = live(conn, ~p"/works/#{work.id}")
  
  assert view
         |> element("[data-action='like']")
         |> render_click() =~ "Liked"
end
```

```bash
# Run component tests
mix test test/venux_web/components/ -v
mix test test/venux_web/live/works_live/show_test.exs -v

# Expected: All tests pass, proper event handling verified
```

### Level 3: Integration Testing  
```bash
# Start the Phoenix server
mix phx.server

# Manual testing checklist:
# 1. Navigate to /works/1 (replace with valid work ID)
# 2. Verify hero section displays with carousel
# 3. Test action buttons (like, favorite, watch now)
# 4. Scroll to 3-column section, verify all components load
# 5. Test responsive design (resize browser window)
# 6. Test on actual mobile device

# Browser dev tools testing:
# - Check for JavaScript console errors
# - Verify image loading performance
# - Test carousel auto-advance functionality
# - Validate accessibility (tab navigation, screen reader compatibility)
```

## Final Validation Checklist
- [ ] All tests pass: `mix test`
- [ ] No compilation warnings: `mix compile`
- [ ] Code quality passes: `mix credo --strict`
- [ ] Hero section displays properly on desktop and mobile
- [ ] Carousel auto-advances with smooth fade transitions  
- [ ] Action buttons respond correctly (like, favorite, watch now)
- [ ] Related works section shows 5 works from same actress
- [ ] Responsive design works across screen sizes
- [ ] Page loads within acceptable time (<3 seconds)
- [ ] Accessibility features work (keyboard navigation, focus states)
- [ ] No console errors in browser dev tools

---

## Anti-Patterns to Avoid
- ❌ Don't create custom JavaScript when CSS animations suffice
- ❌ Don't load all work pictures - limit carousel images for performance
- ❌ Don't ignore mobile experience - design mobile-first
- ❌ Don't break existing functionality (comments, tags, navigation)
- ❌ Don't hardcode image URLs - use existing Venus.Cover patterns
- ❌ Don't skip preloading associations - causes N+1 query problems
- ❌ Don't use sync database calls in LiveView event handlers

## Confidence Score: 8.5/10

**High confidence factors:**
- Clear requirements with specific UI/UX references
- Existing component patterns to follow and reuse
- Well-defined data structures and relationships
- Comprehensive validation strategy
- Progressive implementation approach

**Potential challenges:**
- Carousel performance optimization across browsers
- Complex responsive layout coordination  
- Image loading and caching strategy
- Cross-device animation compatibility

The comprehensive context, clear task breakdown, existing patterns to follow, and thorough validation loops provide strong foundation for successful one-pass implementation.