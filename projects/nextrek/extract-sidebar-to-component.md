name: "Extract Sidebar JavaScript to Component Module"
description: Refactor inline sidebar script from v3 layout to modular JavaScript component

---

## Goal
Extract the inline sidebar toggle script (~140 lines) from `app/views/layouts/v3.html.erb` into a reusable, modular JavaScript component at `app/javascript/components/sidebar_menu.js`, ensuring identical behavior with proper Hotwire Turbo lifecycle management.

## Why
- **Maintainability**: JavaScript logic should not live in .erb template files - harder to lint, test, and version control
- **Reusability**: Encapsulated module can be imported and used across different layouts or contexts
- **Best Practices**: Separation of concerns - keep templates for HTML structure, JavaScript in .js files
- **Testing**: External JS modules are easier to test than inline scripts
- **Build Pipeline**: Take advantage of esbuild bundling, minification, and source maps
- **Turbo Compatibility**: Proper lifecycle management prevents memory leaks and duplicate event listeners during SPA-like navigation

## What
Refactor sidebar menu functionality from inline `<script>` tag to ES6 module while maintaining:
- ✅ Toggle sidebar expand/collapse on desktop
- ✅ Toggle sidebar show/hide on mobile
- ✅ localStorage persistence of sidebar state
- ✅ Responsive behavior (mobile breakpoint at 768px)
- ✅ Resize event handling
- ✅ Graceful degradation when localStorage unavailable
- ✅ **NEW**: Proper Turbo navigation compatibility with cleanup

### Success Criteria
- [ ] No inline `<script>` tag in v3.html.erb (except Turbo-specific initialization if needed)
- [ ] Sidebar component exists at `app/javascript/components/sidebar_menu.js`
- [ ] Component imported and initialized in `app/javascript/v3.js`
- [ ] All original functionality works identically
- [ ] Sidebar state persists across Turbo navigation
- [ ] No duplicate event listeners or memory leaks during navigation
- [ ] No console errors when navigating between pages
- [ ] Build command `yarn build` succeeds without errors
- [ ] Clean, modern ES6+ code with JSDoc comments

## All Needed Context

### Documentation & References
```yaml
- url: https://turbo.hotwired.dev/reference/events
  why: Understanding Turbo lifecycle events (turbo:load, turbo:render, turbo:before-cache) is CRITICAL for proper initialization and cleanup
  key_events:
    - turbo:render: "Fires after Turbo renders the page (use for initialization)"
    - turbo:before-cache: "Fires before Turbo caches current page (use for cleanup)"
    - turbo:load: "Older pattern, prefer turbo:render for 2024"

- url: https://turbo.hotwired.dev/handbook/building
  why: "Event delegation pattern: Avoid attaching listeners directly to elements, use document-level delegation when possible"
  critical: "Turbo Drive cloneNode(true) copies pages, discarding event listeners - must re-attach or use delegation"

- file: app/javascript/forms/BudgetInputForm.js
  why: "Reference pattern for class-based modules in this project"
  pattern: |
    - ES6 class exported with 'export class ClassName'
    - Constructor takes DOM element
    - bindEvents() method for event listener setup
    - Mix of jQuery ($) and vanilla JS is acceptable
    - JSDoc comments for documentation

- file: app/javascript/v3.js
  why: "Entry point where sidebar module will be imported and initialized"
  pattern: |
    - Uses $(document).on("turbo:load", ...) for initialization
    - Imports modules with ES6 import syntax
    - Window-level globals set for libraries (jQuery, toastr, Swal)

- file: app/views/layouts/v3.html.erb (lines 76-217)
  why: "Current inline script to be extracted"
  contains:
    - STORAGE_KEY constant
    - checkMobile(), getSidebarState(), saveSidebarState() utility functions
    - updateSidebar(), expandSidebar(), toggleSidebar() core logic
    - Event listeners: window.resize, button clicks
    - Initialization: updateSidebar() call on page load

- doc: package.json
  why: "Build configuration using esbuild"
  critical: |
    - build command: "esbuild app/javascript/*.js --bundle --sourcemap --outdir=app/assets/builds"
    - Bundles all app/javascript/*.js entry points (including v3.js)
    - Supports ES6 import/export
    - jQuery, @hotwired/turbo-rails already available as dependencies
```

### Current Codebase Structure
```bash
app/
├── javascript/
│   ├── v3.js                      # Entry point - imports and initializes modules
│   ├── v3.css                     # Tailwind CSS entry point
│   ├── controllers/               # Stimulus controllers
│   ├── forms/
│   │   ├── index.js              # Exports all form classes
│   │   └── BudgetInputForm.js    # Example class-based module
│   ├── utils/                     # Utility functions
│   └── reports/                   # Report-specific code
└── views/
    └── layouts/
        └── v3.html.erb            # Contains inline <script> (lines 76-217) ❌
```

### Desired Codebase Structure
```bash
app/
├── javascript/
│   ├── v3.js                      # ✅ Import and initialize sidebar component
│   ├── components/                # ✅ NEW directory for reusable components
│   │   ├── index.js              # ✅ Export all components (sidebar_menu, etc.)
│   │   └── sidebar_menu.js       # ✅ NEW: SidebarMenu class with lifecycle management
│   ├── forms/
│   │   ├── index.js
│   │   └── BudgetInputForm.js
│   ├── utils/
│   └── reports/
└── views/
    └── layouts/
        └── v3.html.erb            # ✅ Remove inline script, clean HTML-only template
```

**File Responsibilities:**
- `components/sidebar_menu.js`: Encapsulates all sidebar toggle logic, state management, event handling, Turbo lifecycle
- `components/index.js`: Exports SidebarMenu class for easy importing
- `v3.js`: Imports sidebar component, initializes on DOMContentLoaded + turbo:render, cleans up on turbo:before-cache

### Known Gotchas & Library Quirks

```javascript
// CRITICAL: Hotwire Turbo Caching Behavior
// When navigating between pages, Turbo caches the current page before loading the next.
// During caching, Turbo uses cloneNode(true) which DISCARDS all event listeners.
// Problem: If you re-attach listeners on every turbo:render without cleanup,
// they accumulate in memory even though they're not functional on the cached page.
//
// Solution Pattern:
// 1. Store references to bound functions as instance properties
// 2. Attach listeners in init()
// 3. Remove listeners in destroy()
// 4. Call destroy() on turbo:before-cache
// 5. Call init() on turbo:render

// CRITICAL: Event Listener Binding
// Cannot remove listeners added as inline arrow functions:
// ❌ BAD: element.addEventListener('click', () => this.method())
// ❌ BAD: element.addEventListener('click', this.method) // 'this' binding lost
// ✅ GOOD: this.boundMethod = this.method.bind(this)
//          element.addEventListener('click', this.boundMethod)
//          element.removeEventListener('click', this.boundMethod) // Works!

// GOTCHA: DOM Element Availability
// Sidebar elements (#sidebar, #sidebarExpand, #sidebarMinimize) must exist in DOM
// before initialization. These elements are rendered by Rails partials in v3.html.erb.
// turbo:render fires AFTER DOM is ready, so safe to query.

// GOTCHA: Resize Listener
// Current implementation resets to responsive defaults on resize (doesn't restore localStorage).
// This is intentional behavior - preserve it in the refactored version.

// PATTERN: localStorage Availability
// Not all browsers/modes support localStorage (private browsing).
// Current code gracefully degrades with try-catch and feature detection.
// Preserve this defensive programming.

// PROJECT CONVENTION: jQuery + Vanilla JS Mix
// Project uses both $ (jQuery) and vanilla querySelector/addEventListener.
// For this component, prefer vanilla JS since we're querying by ID and
// adding simple event listeners. Only use jQuery if complex DOM manipulation needed.
```

## Implementation Blueprint

### Module Architecture

The refactored component follows a **class-based singleton pattern** with explicit lifecycle management:

```javascript
// Pseudocode - sidebar_menu.js structure
export class SidebarMenu {
  constructor() {
    // DOM element references (null until init)
    this.sidebar = null
    this.expandBtn = null
    this.minimizeBtn = null

    // Bound function references for cleanup
    this.boundHandleResize = null
    this.boundHandleExpand = null
    this.boundHandleMinimize = null

    // Constants
    this.STORAGE_KEY = 'nextrek-sidebar-state'
    this.MOBILE_BREAKPOINT = 768
  }

  // Initialize component - query DOM, attach listeners, restore state
  init() {
    // Query DOM elements (return early if not found)
    // Store bound function references
    // Attach event listeners
    // Call updateSidebar() to restore state
  }

  // Cleanup component - remove all event listeners
  destroy() {
    // Remove all event listeners using stored bound references
    // Clear DOM element references
  }

  // Private helper methods (all from current inline script)
  checkMobile() { /* ... */ }
  getSidebarState() { /* ... */ }
  saveSidebarState() { /* ... */ }
  updateSidebar() { /* ... */ }
  expandSidebar() { /* ... */ }
  toggleSidebar() { /* ... */ }
}

// Singleton instance
let sidebarInstance = null

// Export initialization function
export function initSidebar() {
  if (!sidebarInstance) {
    sidebarInstance = new SidebarMenu()
  }
  sidebarInstance.init()
  return sidebarInstance
}

// Export cleanup function
export function cleanupSidebar() {
  if (sidebarInstance) {
    sidebarInstance.destroy()
  }
}
```

### Task List (Sequential Implementation Order)

```yaml
Task 1: Create components directory structure
  CREATE: app/javascript/components/
  CREATE: app/javascript/components/index.js
  WHY: New directory for reusable UI components, following project structure pattern (forms/, utils/, reports/)

Task 2: Create sidebar_menu.js with SidebarMenu class skeleton
  CREATE: app/javascript/components/sidebar_menu.js
  STRUCTURE:
    - Import statements (if needed)
    - SidebarMenu class with constructor
    - Properties: DOM refs, bound functions, constants
    - Methods: init(), destroy() (empty implementations for now)
    - Singleton: sidebarInstance variable
    - Export: initSidebar(), cleanupSidebar(), SidebarMenu class
  WHY: Establish module structure before migrating logic

Task 3: Migrate utility functions to SidebarMenu class methods
  MODIFY: app/javascript/components/sidebar_menu.js
  MIGRATE from v3.html.erb lines 90-146:
    - checkMobile() → this.checkMobile()
    - getSidebarState() → this.getSidebarState()
    - saveSidebarState() → this.saveSidebarState()
  CHANGES:
    - Add 'this.' prefix for accessing this.STORAGE_KEY
    - Add 'this.' prefix for accessing this.sidebar element
    - Preserve all JSDoc comments
    - Keep all error handling and feature detection
  WHY: These are stateless utilities that need access to instance constants

Task 4: Migrate core logic functions to class methods
  MODIFY: app/javascript/components/sidebar_menu.js
  MIGRATE from v3.html.erb lines 152-206:
    - updateSidebar() → this.updateSidebar()
    - expandSidebar() → this.expandSidebar()
    - toggleSidebar() → this.toggleSidebar()
  CHANGES:
    - Add 'this.' prefix for method calls (this.checkMobile(), this.getSidebarState(), etc.)
    - Add 'this.' prefix for DOM element access (this.sidebar)
    - Preserve all logic and comments
  WHY: Core business logic that manipulates sidebar state

Task 5: Implement init() method with DOM query and event binding
  MODIFY: app/javascript/components/sidebar_menu.js → init() method
  LOGIC:
    1. Query DOM elements: this.sidebar, this.expandBtn, this.minimizeBtn
    2. Early return if elements not found (defensive programming)
    3. Create bound function references:
       - this.boundHandleResize = this.handleResize.bind(this)
       - this.boundHandleExpand = this.expandSidebar.bind(this)
       - this.boundHandleMinimize = this.toggleSidebar.bind(this)
    4. Attach event listeners:
       - window.addEventListener('resize', this.boundHandleResize)
       - this.expandBtn.addEventListener('click', this.boundHandleExpand)
       - this.minimizeBtn.addEventListener('click', this.boundHandleMinimize)
    5. Call this.updateSidebar() to initialize state
  ADD NEW METHOD: handleResize() → wrapper that calls updateSidebar()
  WHY: Centralized initialization with proper binding for cleanup

Task 6: Implement destroy() method with event listener cleanup
  MODIFY: app/javascript/components/sidebar_menu.js → destroy() method
  LOGIC:
    1. Remove all event listeners using bound references:
       - window.removeEventListener('resize', this.boundHandleResize)
       - this.expandBtn?.removeEventListener('click', this.boundHandleExpand)
       - this.minimizeBtn?.removeEventListener('click', this.boundHandleMinimize)
    2. Clear DOM references: this.sidebar = null, this.expandBtn = null, etc.
    3. Clear bound function references
  WHY: Prevent memory leaks during Turbo navigation

Task 7: Update components/index.js to export sidebar module
  CREATE/MODIFY: app/javascript/components/index.js
  ADD:
    export { SidebarMenu, initSidebar, cleanupSidebar } from './sidebar_menu'
  WHY: Consistent with project pattern (see forms/index.js)

Task 8: Integrate sidebar component into v3.js entry point
  MODIFY: app/javascript/v3.js
  ADD IMPORT (top of file):
    import { initSidebar, cleanupSidebar } from './components'

  ADD INITIALIZATION (after existing turbo:load handler):
    // Initialize sidebar on initial page load
    document.addEventListener('DOMContentLoaded', () => {
      initSidebar()
    })

    // Re-initialize sidebar after Turbo navigation
    document.addEventListener('turbo:render', () => {
      initSidebar()
    })

    // Cleanup before Turbo caches the page
    document.addEventListener('turbo:before-cache', () => {
      cleanupSidebar()
    })

  WHY: Proper Turbo lifecycle management - init on render, cleanup before cache

Task 9: Remove inline script from v3.html.erb
  MODIFY: app/views/layouts/v3.html.erb
  DELETE: Lines 76-217 (entire <script> tag with sidebar logic)
  VERIFY:
    - Keep surrounding HTML structure
    - Keep other <script> tags if any exist
    - Keep erb tags like <%= javascript_include_tag %>
  WHY: Complete the refactoring - move JS out of template

Task 10: Build and validate
  RUN: yarn build
  EXPECTED: No errors, successful bundle creation
  CHECK: app/assets/builds/v3.js should include bundled sidebar component
  WHY: Ensure esbuild correctly processes the new module structure
```

### Pseudocode for Critical Methods

```javascript
// Task 5 - init() implementation pattern
init() {
  // CRITICAL: Defensive DOM queries - elements might not exist on some pages
  this.sidebar = document.querySelector('#sidebar')
  this.expandBtn = document.querySelector('#sidebarExpand')
  this.minimizeBtn = document.querySelector('#sidebarMinimize')

  if (!this.sidebar || !this.expandBtn || !this.minimizeBtn) {
    console.warn('Sidebar elements not found, skipping initialization')
    return // Early exit - no-op on pages without sidebar
  }

  // PATTERN: Store bound functions for cleanup
  // Cannot use inline arrow functions or removeEventListener won't work
  this.boundHandleResize = this.handleResize.bind(this)
  this.boundHandleExpand = this.expandSidebar.bind(this)
  this.boundHandleMinimize = this.toggleSidebar.bind(this)

  // Attach event listeners with bound references
  window.addEventListener('resize', this.boundHandleResize)
  this.expandBtn.addEventListener('click', this.boundHandleExpand)
  this.minimizeBtn.addEventListener('click', this.boundHandleMinimize)

  // Initialize sidebar state (restore from localStorage or set responsive defaults)
  this.updateSidebar()
}

// Task 6 - destroy() implementation pattern
destroy() {
  // CRITICAL: Must remove listeners to prevent memory leaks during Turbo navigation
  // Use same bound references from init()
  if (this.boundHandleResize) {
    window.removeEventListener('resize', this.boundHandleResize)
  }

  // Optional chaining (?.) for safety - elements might not exist
  if (this.expandBtn && this.boundHandleExpand) {
    this.expandBtn.removeEventListener('click', this.boundHandleExpand)
  }

  if (this.minimizeBtn && this.boundHandleMinimize) {
    this.minimizeBtn.removeEventListener('click', this.boundHandleMinimize)
  }

  // Clear all references for garbage collection
  this.sidebar = null
  this.expandBtn = null
  this.minimizeBtn = null
  this.boundHandleResize = null
  this.boundHandleExpand = null
  this.boundHandleMinimize = null
}

// NEW: handleResize wrapper method
handleResize() {
  // NOTE: Current behavior resets to responsive defaults on resize
  // Does NOT restore from localStorage - this is intentional
  this.updateSidebar()
}

// Task 3 - Example of migrating checkMobile() method
checkMobile() {
  return window.innerWidth < this.MOBILE_BREAKPOINT
}

// Task 3 - Example of migrating getSidebarState() method
getSidebarState() {
  // PATTERN: Defensive programming - check localStorage availability
  if (!window.localStorage) return null

  try {
    const stored = localStorage.getItem(this.STORAGE_KEY) // Note: this.STORAGE_KEY
    if (!stored) return null

    const state = JSON.parse(stored)

    // Validate structure
    if (state && typeof state.ariaExpanded !== 'undefined' && typeof state.dataMinimized !== 'undefined') {
      return state
    }
    return null
  } catch (error) {
    console.warn('Failed to read sidebar state:', error)
    return null
  }
}

// Task 4 - Example of migrating updateSidebar() method
updateSidebar() {
  const savedState = this.getSidebarState() // Note: this.getSidebarState()

  if (savedState) {
    // Restore from localStorage
    this.sidebar.setAttribute('aria-expanded', savedState.ariaExpanded)
    this.sidebar.setAttribute('data-minimized', savedState.dataMinimized)
  } else {
    // Use responsive defaults
    if (this.checkMobile()) { // Note: this.checkMobile()
      this.sidebar.setAttribute('aria-expanded', 'false')
      this.sidebar.setAttribute('data-minimized', 'true')
    } else {
      this.sidebar.setAttribute('aria-expanded', 'true')
      this.sidebar.setAttribute('data-minimized', 'false')
    }
  }
}
```

## Validation Loop

### Level 1: Build Validation (Syntax & Bundling)
```bash
# Run esbuild to verify module syntax and imports
yarn build

# Expected output:
# ✓ app/assets/builds/v3.js  XXX.X kb
# ✓ Success! Built in XXXms

# If errors:
# - Check import/export syntax
# - Verify file paths in import statements
# - Check for syntax errors (missing braces, semicolons)
```

### Level 2: Runtime Validation (Browser Console)
```bash
# Start Rails server
bundle exec rails server

# Navigate to any page using v3 layout
open http://localhost:3000

# Open browser DevTools (F12) → Console tab
# Look for:
# ✅ No JavaScript errors
# ✅ No warnings about sidebar elements not found (unless on page without sidebar)
# ✅ Can see sidebar component initializing (add console.log in init() for debugging)

# If errors:
# - "Cannot read property 'addEventListener' of null" → DOM query failing, check selectors
# - "sidebarInstance is not defined" → Import/export issue, check v3.js imports
# - Multiple initialization logs → turbo:render firing multiple times without cleanup
```

### Level 3: Functional Testing (Manual Test Cases)

#### Test Case 1: Basic Toggle Functionality
1. Navigate to any page with sidebar
2. **Desktop (width > 768px)**:
   - Click expand/collapse button
   - ✅ Sidebar should expand/collapse
   - ✅ aria-expanded attribute should toggle
3. **Mobile (width < 768px)**:
   - Resize browser to mobile size
   - Click minimize/expand button
   - ✅ Sidebar should show/hide
   - ✅ data-minimized attribute should toggle

#### Test Case 2: localStorage Persistence
1. Toggle sidebar to collapsed state
2. Inspect localStorage in DevTools: Application → Local Storage
   - ✅ Key 'nextrek-sidebar-state' should exist
   - ✅ Value should be JSON: `{"ariaExpanded":"false","dataMinimized":"false"}`
3. Refresh page (F5)
   - ✅ Sidebar should remain in collapsed state
4. Clear localStorage and refresh
   - ✅ Sidebar should reset to default (expanded on desktop, minimized on mobile)

#### Test Case 3: Turbo Navigation
1. Toggle sidebar to collapsed state
2. Click any navigation link (Turbo Drive navigation)
3. ✅ Sidebar should remain collapsed on new page
4. Navigate back
5. ✅ Sidebar should still be collapsed
6. Open browser console → check for:
   - ✅ No duplicate event listener warnings
   - ✅ No memory leak warnings

#### Test Case 4: Resize Behavior
1. Start at desktop size with sidebar expanded
2. Resize browser to mobile size
3. ✅ Sidebar should transition to mobile mode
4. Resize back to desktop
5. ✅ Sidebar should transition to desktop mode
6. **Note**: After resize, state resets to responsive defaults (doesn't restore localStorage - this is intentional)

#### Test Case 5: Private Browsing Mode
1. Open browser in private/incognito mode (localStorage may be disabled)
2. Navigate to page with sidebar
3. ✅ No JavaScript errors
4. ✅ Sidebar toggle works normally
5. ✅ State does not persist (graceful degradation)

### Level 4: Code Quality Checks

```bash
# No formal linter configured for JavaScript in this project
# Manual review checklist:

# ✅ All functions have JSDoc comments
# ✅ No console.log statements left in (except strategic console.warn)
# ✅ Consistent indentation (2 spaces)
# ✅ ES6+ features used appropriately (const/let, arrow functions, optional chaining)
# ✅ No jQuery unless necessary (prefer vanilla JS)
# ✅ Error handling present for localStorage operations
# ✅ Defensive null checks before DOM manipulation
```

## Final Validation Checklist

Before marking PRP as complete:

- [ ] **Build succeeds**: `yarn build` completes without errors
- [ ] **No console errors**: Browser console clean on page load
- [ ] **Toggle works**: Desktop expand/collapse functions correctly
- [ ] **Mobile works**: Mobile show/hide functions correctly
- [ ] **Persistence works**: localStorage saves and restores state
- [ ] **Turbo navigation works**: State persists across page transitions
- [ ] **No memory leaks**: Multiple navigations don't cause console warnings
- [ ] **Resize works**: Responsive behavior transitions correctly
- [ ] **Private browsing works**: Graceful degradation when localStorage unavailable
- [ ] **Code is clean**: No inline script remains in v3.html.erb
- [ ] **Code is documented**: JSDoc comments present for all public methods
- [ ] **Pattern matches**: Follows project conventions (class-based, ES6 modules)

---

## Anti-Patterns to Avoid

- ❌ **Don't** use inline arrow functions in addEventListener - they can't be removed
  - ✅ **Do** store bound function references and use them for both add and remove

- ❌ **Don't** forget to call destroy() on turbo:before-cache
  - ✅ **Do** implement full lifecycle with init/destroy and hook into Turbo events

- ❌ **Don't** use jQuery when vanilla JS is sufficient
  - ✅ **Do** use querySelector for ID selectors, addEventListener for events

- ❌ **Don't** assume DOM elements exist without checking
  - ✅ **Do** add defensive null checks and early returns in init()

- ❌ **Don't** ignore Turbo caching behavior
  - ✅ **Do** understand that cloneNode discards listeners, requiring re-initialization

- ❌ **Don't** hardcode magic numbers
  - ✅ **Do** use named constants (this.MOBILE_BREAKPOINT = 768)

- ❌ **Don't** change existing behavior during refactoring
  - ✅ **Do** preserve exact functionality (resize resets to defaults, not localStorage)

- ❌ **Don't** catch all exceptions without logging
  - ✅ **Do** use console.warn for localStorage errors to aid debugging

- ❌ **Don't** initialize on turbo:load if using turbo:render
  - ✅ **Do** use DOMContentLoaded + turbo:render for initialization (2024 best practice)

- ❌ **Don't** create multiple instances of singleton components
  - ✅ **Do** use singleton pattern with module-level instance variable

---

## Confidence Score: **9/10**

### Why 9/10?
- ✅ **Straightforward refactoring**: Moving working code to a module, not rewriting logic
- ✅ **Clear patterns exist**: BudgetInputForm.js provides proven class-based module pattern
- ✅ **Good documentation**: Turbo lifecycle is well-documented at turbo.hotwired.dev
- ✅ **Build system ready**: esbuild already configured, no setup needed
- ✅ **Testable**: Manual test cases are clear and comprehensive

### Risk Factor (-1 point):
- ⚠️ **Turbo edge cases**: While Turbo lifecycle is well-documented, there may be edge cases with caching/restoration that only surface during real-world usage with specific navigation patterns
- ⚠️ **No automated tests**: Relying on manual testing for DOM manipulation - could miss edge cases
- ⚠️ **First component extraction**: This is creating the components/ directory pattern - if issues arise, may need iteration

### Mitigation:
- Comprehensive manual test cases cover common scenarios
- Defensive programming (null checks, try-catch) reduces runtime errors
- Turbo event pattern follows 2024 best practices from official docs
- Can iterate if edge cases discovered in production

---

**Generated**: 2025-10-17
**Project**: Nextrek Accounting SaaS
**Framework**: Ruby on Rails 7.1.5 + Hotwire Turbo + esbuild
