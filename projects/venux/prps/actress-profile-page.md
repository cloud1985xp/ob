# PRP: Actress Profile Page Implementation

## Executive Summary

Implement a comprehensive actress profile page at `/actresses/{actress_id}` featuring a responsive 3-column layout with profile information, infinite-scrolling photo gallery, and integrated chat functionality. The page should provide a Netflix-like user experience with modern, interactive design patterns.

**Confidence Score: 8/10** - High confidence due to comprehensive existing patterns and well-understood data models, with minor uncertainties around mobile UX implementation details.

## Architecture Overview

### Current State
- Route: `/actresses/:id` currently handled by `VenusWeb.ActressController.show/2`
- Template: `lib/venux_web/controllers/actress_html/show.html.heex` (basic layout)
- Data: `Venus.Actresses.find/1` and `Venus.Actresses.list_works/2` available

### Target State
- Convert to LiveView: `VenusWeb.ActressesLive.Show`
- 3-column responsive layout with infinite scroll
- Reusable components: `work_card.ex`, `photo_card.ex`, `photo_list.ex`
- Mobile-optimized with floating sidebar

## Data Models & Context

### Core Schemas
```elixir
# Venus.Actresses.Actress
schema "actresses" do
  field :avatar, Venus.Avatar.Type
  field :name, :string
  field :display_name, :string
  field :description, :string
  field :rank, :integer
  # ... other fields
end

# Venus.Actresses.ActressWork  
schema "actress_works" do
  field :actress_id, :integer
  field :title, :string
  field :cover, Venus.Cover.Type
  field :description, :string
  field :published_on, :date
  # ... other fields
end

# Venus.Actresses.ActressWorkPicture
schema "actress_work_pictures" do
  field :work_id, :integer
  field :file, Venus.Picture.Type  # This contains the photo URL
  field :width, :integer
  field :height, :integer
  field :ratio, :float
  # ... other fields
end
```

### Data Relationships
```
actress (1) -> (many) actress_works (1) -> (many) actress_work_pictures
```

**Critical**: Photos are accessed via `actress -> works -> work_pictures`, not directly from actress.

## Implementation Blueprint

### Phase 1: Convert Controller to LiveView

1. **Update Router** (`lib/venux_web/router.ex`)
```elixir
# Replace this line:
# get "/actresses/:id", ActressController, :show

# With this:
live "/actresses/:id", ActressesLive.Show, :show
```

2. **Create LiveView Module** (`lib/venux_web/live/actresses_live/show.ex`)
```elixir
defmodule VenusWeb.ActressesLive.Show do
  use VenusWeb, :live_view
  
  alias Venus.Actresses
  alias VenusWeb.ActressChat

  @impl true
  def mount(_params, _session, socket) do
    socket = 
      socket
      |> assign(page_title: "Actress Profile")
      |> assign(show_chat: false)
      |> assign(chat_messages: [])
      |> assign(photos_page: 1)
      |> assign(has_more_photos: true)
      |> assign(loading_photos: false)
      |> assign(show_mobile_sidebar: false)
    
    {:ok, socket}
  end

  @impl true  
  def handle_params(%{"id" => actress_id}, _uri, socket) do
    actress = Actresses.find(actress_id)
    latest_works = Actresses.list_works(actress, %{"page" => 1, "per_page" => 5})
    
    # Load initial photos
    initial_photos = load_actress_photos(actress, 1)
    
    socket = 
      socket
      |> assign(actress: actress)
      |> assign(latest_works: latest_works.entries)
      |> assign(page_title: actress.display_name)
      |> stream(:photos, initial_photos.entries)
      |> assign(has_more_photos: length(initial_photos.entries) == 30)
    
    {:noreply, socket}
  end

  # Photo loading function
  defp load_actress_photos(actress, page) do
    # Query photos through works relationship
    from(p in Venus.Actresses.ActressWorkPicture,
      join: w in Venus.Actresses.ActressWork, on: p.work_id == w.id,
      where: w.actress_id == ^actress.id,
      order_by: [desc: p.inserted_at],
      select: %{
        id: p.id,
        file: p.file,
        ratio: p.ratio,
        width: p.width,
        height: p.height,
        work_title: w.title,
        work_id: w.id
      }
    )
    |> Venus.Repo.paginate(%{"page" => page, "per_page" => 30})
  end

  @impl true
  def handle_event("load_more_photos", _params, socket) do
    if socket.assigns.has_more_photos and not socket.assigns.loading_photos do
      next_page = socket.assigns.photos_page + 1
      photos_page = load_actress_photos(socket.assigns.actress, next_page)
      
      socket = 
        socket
        |> assign(photos_page: next_page)
        |> assign(has_more_photos: length(photos_page.entries) == 30)
        |> assign(loading_photos: false)
        |> stream(:photos, photos_page.entries, at: -1)
      
      {:noreply, socket}
    else
      {:noreply, socket}
    end
  end

  def handle_event("toggle_mobile_sidebar", _params, socket) do
    {:noreply, assign(socket, show_mobile_sidebar: not socket.assigns.show_mobile_sidebar)}
  end

  def handle_event("toggle_chat", _params, socket) do
    {:noreply, assign(socket, show_chat: not socket.assigns.show_chat)}
  end

  # Add other event handlers for photo interactions
end
```

### Phase 2: Create Reusable Components

3. **Work Card Component** (`lib/venux_web/components/work_card.ex`)
```elixir
defmodule VenusWeb.WorkCard do
  use Phoenix.Component
  alias VenusWeb.UI

  attr :work, :map, required: true
  attr :class, :string, default: ""
  attr :show_actress_info, :boolean, default: false

  def work_card(assigns) do
    ~H"""
    <div class={["group cursor-pointer", @class]}>
      <div class="relative bg-white/70 backdrop-blur-sm border border-rose-200/30 rounded-2xl overflow-hidden transition-all duration-300 group-hover:scale-105 group-hover:shadow-2xl group-hover:shadow-rose-500/20">
        <!-- Work Cover -->
        <div class="relative h-48 bg-gray-100">
          <%= if @work.cover do %>
            <UI.lazy_img 
              src={Venus.Cover.url({@work.cover, @work}, :title)}
              alt={@work.title}
              class="w-full h-full object-cover"
            />
          <% else %>
            <div class="w-full h-full flex items-center justify-center">
              <svg class="w-12 h-12 text-gray-600" fill="currentColor" viewBox="0 0 24 24">
                <path d="M14,2H6A2,2 0 0,0 4,4V20A2,2 0 0,0 6,22H18A2,2 0 0,0 20,20V8L14,2M18,20H6V4H13V9H18V20Z"/>
              </svg>
            </div>
          <% end %>

          <!-- Hover Actions -->
          <div class="absolute inset-0 bg-black/0 group-hover:bg-black/70 transition-all duration-300 flex items-center justify-center">
            <div class="opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex space-x-3">
              <button
                phx-click="view_work"
                phx-value-work_id={@work.id}
                class="w-12 h-12 bg-gradient-to-r from-red-500 to-pink-500 text-white rounded-full flex items-center justify-center transition-all duration-300 shadow-lg hover:scale-110"
              >
                <svg class="w-6 h-6" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M8 5v14l11-7z"/>
                </svg>
              </button>
            </div>
          </div>
        </div>

        <!-- Work Info -->
        <div class="p-4">
          <h3 class="text-lg font-semibold text-gray-800 mb-2 line-clamp-2">{@work.title}</h3>
          <p class="text-gray-600 text-sm mb-3 line-clamp-2">{@work.description}</p>
          <div class="flex justify-between items-center text-xs text-gray-500">
            <span>{Calendar.strftime(@work.published_on, "%b %d, %Y")}</span>
          </div>
        </div>
      </div>
    </div>
    """
  end
end
```

4. **Photo Card Component** (`lib/venux_web/components/photo_card.ex`)
```elixir
defmodule VenusWeb.PhotoCard do
  use Phoenix.Component
  alias VenusWeb.UI

  attr :photo, :map, required: true
  attr :class, :string, default: ""

  def photo_card(assigns) do
    ~H"""
    <div class={["break-inside-avoid group cursor-pointer", @class]}>
      <div class="relative bg-white/70 backdrop-blur-sm border border-purple-200/30 rounded-2xl overflow-hidden transition-all duration-300 group-hover:scale-105 group-hover:shadow-2xl group-hover:shadow-purple-500/20">
        <!-- Photo -->
        <div class="relative">
          <UI.lazy_img 
            src={Venus.Picture.url({@photo.file, @photo}, :original)}
            alt={@photo.work_title || "Actress photo"}
            class="w-full h-auto object-cover"
            style={"aspect-ratio: #{@photo.ratio}"}
          />

          <!-- Hover Actions -->
          <div class="absolute inset-0 bg-black/0 group-hover:bg-black/60 transition-all duration-300 flex items-center justify-center">
            <div class="opacity-0 group-hover:opacity-100 transition-opacity duration-300 flex space-x-3">
              <button
                phx-click="view_photo"
                phx-value-photo_id={@photo.id}
                class="w-12 h-12 bg-gradient-to-r from-purple-500 to-blue-500 text-white rounded-full flex items-center justify-center transition-all duration-300 shadow-lg hover:scale-110"
              >
                <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"></path>
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"></path>
                </svg>
              </button>
              <button
                phx-click="like_photo"
                phx-value-photo_id={@photo.id}
                class="w-12 h-12 bg-white/80 border border-purple-200 text-gray-700 rounded-full flex items-center justify-center transition-all duration-300 shadow-lg hover:scale-110"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"></path>
                </svg>
              </button>
              <button
                phx-click="share_photo"
                phx-value-photo_id={@photo.id}
                class="w-12 h-12 bg-white/80 border border-purple-200 text-gray-700 rounded-full flex items-center justify-center transition-all duration-300 shadow-lg hover:scale-110"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8.684 13.342C8.886 12.938 9 12.482 9 12c0-.482-.114-.938-.316-1.342m0 2.684a3 3 0 110-2.684m0 2.684l6.632 3.316m-6.632-6l6.632-3.316m0 0a3 3 0 105.367-2.684 3 3 0 00-5.367 2.684zm0 9.316a3 3 0 105.367 2.684 3 3 0 00-5.367-2.684z"></path>
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    """
  end
end
```

5. **Main Template** (`lib/venux_web/live/actresses_live/show.html.heex`)
```heex
<div class="min-h-screen bg-gray-900">
  <!-- Mobile Header -->
  <div class="lg:hidden flex items-center justify-between p-4 border-b border-gray-700">
    <h1 class="text-xl font-semibold text-white">{@actress.display_name}</h1>
    <button 
      phx-click="toggle_mobile_sidebar"
      class="text-gray-400 hover:text-white"
    >
      <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
      </svg>
    </button>
  </div>

  <div class="flex h-screen">
    <!-- Left Column: Profile & Works (fixed on large, hidden on mobile) -->
    <div class="hidden lg:block w-80 p-6 border-r border-gray-700 overflow-y-auto">
      <!-- Profile Section -->
      <div class="mb-8">
        <div class="relative mb-6">
          <!-- Cover Background -->
          <%= if @actress.avatar do %>
            <div class="h-32 bg-gradient-to-r from-purple-600 to-blue-600 rounded-2xl relative overflow-hidden">
              <UI.lazy_img 
                src={Venus.Avatar.url({@actress.avatar, @actress}, :original)}
                class="w-full h-full object-cover opacity-30"
              />
            </div>
          <% else %>
            <div class="h-32 bg-gradient-to-r from-purple-600 to-blue-600 rounded-2xl"></div>
          <% end %>
          
          <!-- Avatar -->
          <div class="absolute -bottom-8 left-6">
            <div class="w-20 h-20 rounded-full border-4 border-gray-800 overflow-hidden bg-gray-700">
              <%= if @actress.avatar do %>
                <UI.lazy_img 
                  src={Venus.Avatar.url({@actress.avatar, @actress}, :thumb)}
                  alt={@actress.display_name}
                  class="w-full h-full object-cover"
                />
              <% else %>
                <div class="w-full h-full flex items-center justify-center">
                  <svg class="w-10 h-10 text-gray-400" fill="currentColor" viewBox="0 0 24 24">
                    <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
                  </svg>
                </div>
              <% end %>
            </div>
          </div>
        </div>

        <div class="mt-4 pl-6">
          <h1 class="text-2xl font-bold text-white mb-2">{@actress.display_name}</h1>
          <p class="text-gray-400 mb-4">{@actress.description}</p>
          
          <!-- Stats -->
          <div class="flex space-x-6 text-sm">
            <div class="text-center">
              <div class="text-xl font-bold text-white">{@actress.rank}</div>
              <div class="text-gray-400">Rank</div>
            </div>
            <div class="text-center">
              <div class="text-xl font-bold text-white">{length(@latest_works)}</div>
              <div class="text-gray-400">Works</div>
            </div>
          </div>
        </div>
      </div>

      <!-- Latest Works Section -->
      <div>
        <h2 class="text-lg font-semibold text-white mb-4">Latest Works</h2>
        <div class="space-y-4">
          <%= for work <- @latest_works do %>
            <VenusWeb.WorkCard.work_card work={work} class="w-full" />
          <% end %>
        </div>
      </div>
    </div>

    <!-- Middle Column: Photos (scrollable) -->
    <div class="flex-1 p-6 overflow-y-auto">
      <h2 class="text-2xl font-bold text-white mb-6 lg:block hidden">Photos</h2>
      
      <!-- Photos Grid with Infinite Scroll -->
      <div 
        id="photos-grid"
        phx-update="stream"
        phx-viewport-bottom={@has_more_photos && JS.push("load_more_photos")}
        class="columns-1 sm:columns-2 md:columns-3 lg:columns-4 gap-4 space-y-4"
      >
        <%= for {id, photo} <- @streams.photos do %>
          <div id={id}>
            <VenusWeb.PhotoCard.photo_card photo={photo} />
          </div>
        <% end %>
      </div>

      <!-- Loading State -->
      <%= if @loading_photos do %>
        <div class="flex justify-center items-center py-8">
          <div class="w-8 h-8 border-4 border-gray-700 border-t-purple-500 rounded-full animate-spin"></div>
        </div>
      <% end %>
    </div>

    <!-- Right Column: Portrait & Chat (fixed on large, floating on mobile) -->
    <div class={[
      "w-80 border-l border-gray-700 flex flex-col",
      if(@show_mobile_sidebar, 
        do: "lg:relative fixed inset-y-0 right-0 z-50 bg-gray-900", 
        else: "hidden lg:flex"
      )
    ]}>
      <!-- Close button for mobile -->
      <%= if @show_mobile_sidebar do %>
        <div class="lg:hidden flex justify-end p-4">
          <button 
            phx-click="toggle_mobile_sidebar"
            class="text-gray-400 hover:text-white"
          >
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"></path>
            </svg>
          </button>
        </div>
      <% end %>

      <!-- Portrait Section -->
      <div class="p-6 border-b border-gray-700">
        <div class="aspect-square rounded-2xl overflow-hidden bg-gray-800 mb-4">
          <%= if @actress.avatar do %>
            <UI.lazy_img 
              src={Venus.Avatar.url({@actress.avatar, @actress}, :original)}
              alt={@actress.display_name}
              class="w-full h-full object-cover"
            />
          <% else %>
            <div class="w-full h-full flex items-center justify-center">
              <svg class="w-20 h-20 text-gray-600" fill="currentColor" viewBox="0 0 24 24">
                <path d="M12 12c2.21 0 4-1.79 4-4s-1.79-4-4-4-4 1.79-4 4 1.79 4 4 4zm0 2c-2.67 0-8 1.34-8 4v2h16v-2c0-2.66-5.33-4-8-4z"/>
              </svg>
            </div>
          <% end %>
        </div>
        
        <button 
          phx-click="toggle_chat"
          class="w-full bg-gradient-to-r from-red-500 to-pink-500 hover:from-red-600 hover:to-pink-600 text-white font-semibold py-3 rounded-xl transition-all duration-300"
        >
          💬 Chat with {@actress.display_name}
        </button>
      </div>

      <!-- Chat Section -->
      <div class="flex-1 flex flex-col">
        <%= if @show_chat do %>
          <VenusWeb.ActressChat.chat_modal 
            actress={@actress} 
            show={@show_chat} 
            messages={@chat_messages} 
          />
        <% else %>
          <div class="flex-1 flex items-center justify-center text-gray-500">
            <div class="text-center">
              <svg class="w-16 h-16 mx-auto mb-4 opacity-50" fill="currentColor" viewBox="0 0 24 24">
                <path d="M20 2H4c-1.1 0-2 .9-2 2v12c0 1.1.9 2 2 2h4l4 4 4-4h4c1.1 0 2-.9 2-2V4c0-1.1-.9-2-2-2zm-2 12H6v-2h12v2zm0-3H6V9h12v2zm0-3H6V6h12v2z"/>
              </svg>
              <p class="text-sm">Click above to start chatting!</p>
            </div>
          </div>
        <% end %>
      </div>
    </div>
  </div>

  <!-- Mobile Sidebar Backdrop -->
  <%= if @show_mobile_sidebar do %>
    <div 
      class="lg:hidden fixed inset-0 bg-black/50 z-40"
      phx-click="toggle_mobile_sidebar"
    ></div>
  <% end %>
</div>
```

## Responsive Design Strategy

### Breakpoint Strategy
- **Mobile (< 1024px)**: Single column with floating sidebar
- **Large (>= 1024px)**: Three-column fixed layout

### Key Responsive Patterns
```css
/* Grid responsive behavior */
.columns-1 sm:columns-2 md:columns-3 lg:columns-4

/* Sidebar behavior */
.hidden lg:block /* Left sidebar */
.hidden lg:flex /* Right sidebar */

/* Mobile overlay */
.fixed inset-y-0 right-0 z-50 /* Mobile right sidebar */
```

## External Documentation & References

### Phoenix LiveView Infinite Scroll
- **Primary Reference**: https://www.yellowduck.be/posts/scroll-events-and-infinite-pagination-in-phoenix-liveview
- **Official Docs**: https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#stream/3
- **Stream Implementation**: Use `phx-viewport-bottom` with `phx-update="stream"`
- **Medium Article**: https://brooklinmyers.medium.com/pagination-and-infinite-scroll-in-phoenix-d2a5f9bac5d6

### Code Patterns to Follow
- **Infinite Scroll**: `lib/venux_web/live/actresses_live/index.ex` (lines 46-69)
- **Component Structure**: `lib/venux_web/components/photo_gallery.ex`
- **Responsive Layout**: `lib/venux_web/components/shared_sidebar.ex`
- **Image Handling**: `lib/venux_web/ui.ex` (lazy_img component)

## Validation Gates

### Compilation & Syntax
```bash
mix compile
```

### Code Quality
```bash
mix credo
```

### Testing
```bash
mix test
```

### Manual Verification Checklist
- [ ] Route `/actresses/{actress_id}` loads without errors
- [ ] Profile section displays actress info correctly
- [ ] Latest works section shows 5 most recent works
- [ ] Photos load in masonry grid layout
- [ ] Infinite scroll loads 30 photos per batch
- [ ] Mobile sidebar toggles correctly
- [ ] Chat integration works
- [ ] All images use lazy loading
- [ ] Responsive design works on mobile/desktop
- [ ] Work cards and photo cards are reusable components

## Risk Mitigation & Common Pitfalls

### Data Relationship Issues
**Pitfall**: Trying to query photos directly from actress
**Solution**: Use proper join: `actress -> actress_works -> actress_work_pictures`

### Image URL Generation
**Pitfall**: Incorrect image URL format
**Solution**: Use `Venus.Picture.url({photo.file, photo}, :original)` pattern

### Stream Implementation
**Pitfall**: Missing DOM IDs or improper stream setup
**Solution**: Ensure `phx-update="stream"` and unique IDs for each photo

### Mobile UX
**Pitfall**: Sidebar not properly responsive
**Solution**: Use Tailwind's responsive utilities and proper z-index layering

### Performance
**Pitfall**: Loading too many photos at once
**Solution**: Stick to batch size of 30, implement proper loading states

## Implementation Tasks Checklist

1. [ ] Update router to use LiveView
2. [ ] Create `VenusWeb.ActressesLive.Show` module
3. [ ] Implement photo loading query with proper joins
4. [ ] Create `VenusWeb.WorkCard` component
5. [ ] Create `VenusWeb.PhotoCard` component  
6. [ ] Create responsive template with 3-column layout
7. [ ] Implement infinite scroll with `phx-viewport-bottom`
8. [ ] Add mobile sidebar toggle functionality
9. [ ] Integrate existing chat component
10. [ ] Test responsive behavior on multiple screen sizes
11. [ ] Verify all validation gates pass
12. [ ] Test infinite scroll with real data

## Success Criteria

- **Functional**: All features work as specified in INITIAL.md
- **Responsive**: Seamless experience on mobile and desktop
- **Performance**: Photos load efficiently with infinite scroll
- **Reusable**: Components can be used in other parts of the application
- **Maintainable**: Code follows existing patterns and conventions
- **Tested**: All validation gates pass successfully

**Estimated Implementation Time**: 6-8 hours for experienced Phoenix developer

**Confidence Score: 8/10** - Strong foundation with existing patterns, minor uncertainty around mobile UX details and exact image URL formats.