name: "Sidebar State Persistence with localStorage"
description: |
  Add localStorage persistence to v3 layout sidebar to remember expanded/collapsed state across page refreshes and navigation.

## Purpose
Enhance user experience by persisting sidebar state (expanded/collapsed) across page refreshes and navigation using browser localStorage, without modifying backend code or HTML structure.

## Core Principles
1. **Context is King**: Include ALL necessary documentation, examples, and caveats
2. **Validation Loops**: Provide executable tests/lints the AI can run and fix
3. **Information Dense**: Use keywords and patterns from the codebase
4. **Progressive Success**: Start simple, validate, then enhance
5. **Global rules**: Be sure to follow all rules in CLAUDE.md

---

## Goal
Improve the sidebar menu in the v3 layout to remember its last state (expanded or collapsed) when users refresh the page or navigate to another page, using browser localStorage for state persistence.

## Why
- **User Experience**: Users shouldn't lose their sidebar preferences on every page refresh
- **Consistency**: State should persist across navigation within the application
- **No Backend Changes**: Pure client-side solution requiring no server modifications
- **Modern Best Practice**: localStorage is the standard approach for client-side state persistence

## What
Modify the JavaScript in `app/views/layouts/v3.html.erb` (lines 76-124) to:
1. Read sidebar state from localStorage on page load
2. Save sidebar state to localStorage when user toggles the sidebar
3. Maintain existing toggle functionality without breaking current behavior
4. Handle edge cases (localStorage unavailable, corrupt data, etc.)
5. Support both mobile and desktop responsive behaviors

### Success Criteria
- [x] Sidebar state persists across page refreshes
- [x] Sidebar state persists across Turbo navigation
- [x] Original toggle functionality remains intact
- [x] Mobile/desktop responsive behavior unchanged
- [x] Code includes clear comments explaining the logic
- [x] Works in all modern browsers (Chrome, Firefox, Safari, Edge)
- [x] Handles localStorage unavailability gracefully (private browsing mode)
- [x] No backend or HTML structure changes

## All Needed Context

### Documentation & References

```yaml
# MUST READ - Include these in your context window

- url: https://blog.logrocket.com/localstorage-javascript-complete-guide/
  why: Comprehensive guide on localStorage usage, best practices
  critical: Always use try-catch for JSON.parse/stringify, handle localStorage not available

- url: https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage
  why: Official MDN documentation for localStorage API
  critical: localStorage only stores strings, quota limits (5-10MB), synchronous operations

- file: app/views/layouts/v3.html.erb
  why: Current sidebar toggle implementation (lines 76-124)
  critical: |
    - Uses aria-expanded attribute for desktop state (true/false)
    - Uses data-minimized attribute for mobile state (true/false)
    - checkMobile() determines viewport < 768px
    - updateSidebar() sets initial state based on screen size
    - expandSidebar() handles desktop toggle
    - toggleSidebar() handles mobile toggle

- file: app/views/layouts/v3/navigation/_normal.html.erb
  why: Sidebar HTML structure with key attributes
  critical: |
    - Sidebar ID: #sidebar
    - Default attributes: aria-expanded="true", data-minimized="false"
    - Responsive classes handle mobile/desktop display
    - Toggle buttons: #sidebarExpand, #sidebarMinimize

- file: app/javascript/v3.js
  why: JavaScript patterns used in the project
  critical: |
    - Modern ES6+ syntax (const, arrow functions, template literals)
    - Turbo Rails for navigation (@hotwired/turbo-rails)
    - jQuery available but prefer vanilla JS
    - Clean, modern coding style

- pattern: No existing localStorage usage in codebase
  why: First implementation - establish pattern for future use
  critical: Set precedent for error handling and code organization
```

### Current Codebase Structure

```
app/
├── views/
│   └── layouts/
│       ├── v3.html.erb                    # Target file - contains script to modify
│       └── v3/
│           ├── navigation/
│           │   └── _normal.html.erb       # Sidebar HTML structure
│           ├── _topbar.html.erb
│           └── _footer.html.erb
└── javascript/
    ├── v3.js                              # Main v3 JS entry point (reference for patterns)
    ├── utils/
    │   └── index.js                       # Utility functions (reference for patterns)
    └── controllers/                       # Stimulus controllers
```

### Current Implementation (v3.html.erb lines 76-124)

```javascript
<script>
  const sidebarExpandBtn = document.querySelector("#sidebarExpand");
  const sidebarMinimizeBtn = document.querySelector("#sidebarMinimize");
  const sidebar = document.querySelector("#sidebar");
  const topbar = document.querySelector("#topbar");

  function checkMobile() {
    return window.innerWidth < 768;
  }

  function updateSidebar() {
    if (checkMobile()) {
      // Mobile: hide sidebar initially
      sidebar.setAttribute("aria-expanded", "false");
      sidebar.setAttribute("data-minimized", "true");
    } else {
      // Desktop: show expanded sidebar
      sidebar.setAttribute("aria-expanded", "true");
      sidebar.setAttribute("data-minimized", "false");
    }
  }

  const expandSidebar = () => {
    if (checkMobile()) {
      toggleSidebar();
      return;
    }

    // Desktop: toggle aria-expanded
    const isExpanded = sidebar.getAttribute("aria-expanded") === "true";
    sidebar.setAttribute("aria-expanded", !isExpanded);
  };

  function toggleSidebar() {
    if (!checkMobile()) return;

    // Mobile: toggle data-minimized
    const isMinimized = sidebar.dataset.minimized === "true";
    sidebar.setAttribute("data-minimized", !isMinimized);
  }

  // Initialize
  updateSidebar();

  // Event listeners
  window.addEventListener("resize", updateSidebar);
  sidebarMinimizeBtn.addEventListener("click", toggleSidebar);
  sidebarExpandBtn.addEventListener("click", expandSidebar);
</script>
```

### Known Gotchas & Library Quirks

```javascript
// CRITICAL: localStorage considerations
// 1. localStorage only stores strings - use JSON.parse/stringify
// 2. localStorage may not be available (private browsing, security settings)
// 3. Always wrap in try-catch to handle errors gracefully
// 4. localStorage is synchronous - avoid storing large data
// 5. Check for existence before reading to avoid errors

// CRITICAL: Turbo Rails navigation
// - Turbo caches pages and may restore from cache
// - Need to re-apply sidebar state on turbo:load events
// - Script runs on every page load/navigation

// CRITICAL: Responsive behavior
// - Mobile/desktop states are separate (aria-expanded vs data-minimized)
// - Screen resize should NOT restore from localStorage (preserve current behavior)
// - Only restore from localStorage on initial page load/navigation

// CRITICAL: Sidebar attributes
// - aria-expanded: Controls desktop expansion (true/false string)
// - data-minimized: Controls mobile visibility (true/false string)
// - Both are string values, not booleans

// CRITICAL: Browser compatibility
// - localStorage available in all modern browsers
// - Use feature detection, not browser detection
// - Provide fallback if localStorage unavailable
```

## Implementation Blueprint

### localStorage Structure

```javascript
// Storage key
const STORAGE_KEY = 'nextrek-sidebar-state';

// Storage value structure (JSON stringified)
{
  ariaExpanded: "true",      // Desktop state (string "true" or "false")
  dataMinimized: "false"     // Mobile state (string "true" or "false")
}
```

### Task List

```yaml
Task 1: Create localStorage utility functions
  description: Add helper functions to safely read/write sidebar state
  location: Inside <script> tag in app/views/layouts/v3.html.erb
  actions:
    - CREATE getSidebarState() function
      - Check if localStorage is available
      - Try to read and parse stored state
      - Return parsed state or null if unavailable/corrupted
      - Use try-catch for error handling

    - CREATE saveSidebarState() function
      - Check if localStorage is available
      - Stringify and save current sidebar attributes
      - Use try-catch for error handling
      - Fail silently if unavailable

Task 2: Modify updateSidebar() to restore from localStorage
  description: Load saved state on page load instead of always resetting
  location: app/views/layouts/v3.html.erb, updateSidebar() function
  actions:
    - FIND: updateSidebar() function
    - MODIFY logic:
      - First, try to load state from localStorage
      - If state exists, apply it to sidebar
      - If no state, use current default logic (mobile/desktop defaults)
      - Only restore state, don't trigger save
    - PRESERVE: Screen size detection logic for new users
    - KEEP: Attribute setting patterns

Task 3: Add localStorage save to expandSidebar()
  description: Save state when user toggles on desktop
  location: app/views/layouts/v3.html.erb, expandSidebar() function
  actions:
    - FIND: expandSidebar() function
    - INJECT: Call saveSidebarState() after setAttribute
    - PRESERVE: Existing toggle logic
    - ENSURE: Save happens after state change

Task 4: Add localStorage save to toggleSidebar()
  description: Save state when user toggles on mobile
  location: app/views/layouts/v3.html.erb, toggleSidebar() function
  actions:
    - FIND: toggleSidebar() function
    - INJECT: Call saveSidebarState() after setAttribute
    - PRESERVE: Existing toggle logic
    - ENSURE: Save happens after state change

Task 5: Modify resize event handler
  description: Ensure window resize doesn't overwrite user preference
  location: app/views/layouts/v3.html.erb, resize event listener
  actions:
    - REVIEW: Current resize handler calls updateSidebar()
    - DECISION: Keep current behavior (resize resets to defaults)
    - RATIONALE: When switching mobile/desktop, responsive defaults make sense
    - ALTERNATIVE: Could save after resize, but may be confusing
    - KEEP: Current implementation unchanged

Task 6: Add comprehensive code comments
  description: Document the localStorage logic for maintainability
  location: Throughout the modified script
  actions:
    - ADD: Function-level comments explaining purpose
    - ADD: Inline comments for complex logic
    - ADD: Error handling explanations
    - STYLE: Clear, concise, professional comments
```

### Pseudocode for Key Functions

```javascript
// Task 1: localStorage Utility Functions

/**
 * Safely retrieve sidebar state from localStorage
 * @returns {Object|null} Parsed sidebar state or null if unavailable
 */
function getSidebarState() {
  // Check if localStorage is available (feature detection)
  if (!window.localStorage) return null;

  try {
    // Attempt to read from localStorage
    const stored = localStorage.getItem(STORAGE_KEY);
    if (!stored) return null;

    // Parse JSON string to object
    const state = JSON.parse(stored);

    // Validate structure (has required properties)
    if (state && typeof state.ariaExpanded !== 'undefined' && typeof state.dataMinimized !== 'undefined') {
      return state;
    }
    return null;
  } catch (error) {
    // Handle JSON parse errors or localStorage exceptions
    // Fail silently - return null to use defaults
    console.warn('Failed to read sidebar state:', error);
    return null;
  }
}

/**
 * Save current sidebar state to localStorage
 */
function saveSidebarState() {
  // Check if localStorage is available
  if (!window.localStorage) return;

  try {
    // Get current state from DOM attributes
    const state = {
      ariaExpanded: sidebar.getAttribute("aria-expanded"),
      dataMinimized: sidebar.getAttribute("data-minimized")
    };

    // Stringify and save
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  } catch (error) {
    // Handle quota exceeded or localStorage unavailable
    // Fail silently - feature degrades gracefully
    console.warn('Failed to save sidebar state:', error);
  }
}

// Task 2: Modified updateSidebar()

function updateSidebar() {
  // Try to restore saved state first
  const savedState = getSidebarState();

  if (savedState) {
    // Restore from localStorage
    sidebar.setAttribute("aria-expanded", savedState.ariaExpanded);
    sidebar.setAttribute("data-minimized", savedState.dataMinimized);
  } else {
    // No saved state - use responsive defaults
    if (checkMobile()) {
      sidebar.setAttribute("aria-expanded", "false");
      sidebar.setAttribute("data-minimized", "true");
    } else {
      sidebar.setAttribute("aria-expanded", "true");
      sidebar.setAttribute("data-minimized", "false");
    }
  }
}

// Task 3: Modified expandSidebar()

const expandSidebar = () => {
  if (checkMobile()) {
    toggleSidebar();
    return;
  }

  // Desktop: toggle aria-expanded
  const isExpanded = sidebar.getAttribute("aria-expanded") === "true";
  sidebar.setAttribute("aria-expanded", !isExpanded);

  // Save state after toggle
  saveSidebarState();
};

// Task 4: Modified toggleSidebar()

function toggleSidebar() {
  if (!checkMobile()) return;

  // Mobile: toggle data-minimized
  const isMinimized = sidebar.dataset.minimized === "true";
  sidebar.setAttribute("data-minimized", !isMinimized);

  // Save state after toggle
  saveSidebarState();
}
```

### Integration Points

```yaml
FILE_MODIFICATION:
  file: app/views/layouts/v3.html.erb
  section: <script> tag (lines 76-124)
  changes:
    - ADD: STORAGE_KEY constant at top
    - ADD: getSidebarState() function
    - ADD: saveSidebarState() function
    - MODIFY: updateSidebar() to restore from localStorage
    - MODIFY: expandSidebar() to save after toggle
    - MODIFY: toggleSidebar() to save after toggle
    - ADD: Comments throughout for clarity

NO_OTHER_CHANGES:
  - No backend modifications
  - No HTML structure changes
  - No CSS changes
  - No other JavaScript files affected
```

## Validation Loop

### Level 1: Code Review & Syntax Check
```bash
# Manual review checklist:
# 1. Verify JavaScript syntax is valid (no console errors)
# 2. Check that all original functionality preserved
# 3. Ensure comments are clear and helpful
# 4. Confirm no HTML/backend changes made

# Open browser console and check for errors
# Navigate to any page using v3 layout
# Expected: No JavaScript errors in console
```

### Level 2: Functional Testing
```bash
# Test Case 1: Desktop - Expand/Collapse Persistence
# 1. Open application in desktop viewport (width > 768px)
# 2. Toggle sidebar to collapsed using expand button
# 3. Refresh page (Cmd+R / Ctrl+R)
# Expected: Sidebar remains collapsed after refresh

# Test Case 2: Mobile - Show/Hide Persistence
# 1. Open application in mobile viewport (width < 768px)
# 2. Toggle sidebar to visible using toggle button
# 3. Refresh page
# Expected: Sidebar remains visible after refresh

# Test Case 3: Turbo Navigation
# 1. Set sidebar to collapsed state
# 2. Navigate to different page using internal link (Turbo navigation)
# 3. Observe sidebar state
# Expected: Sidebar remains collapsed after navigation

# Test Case 4: localStorage Unavailable (Private Browsing)
# 1. Open browser in private/incognito mode
# 2. Toggle sidebar multiple times
# 3. Refresh page
# Expected: Sidebar works normally, gracefully degrades (uses defaults)

# Test Case 5: Cross-Tab Consistency
# 1. Open application in two tabs
# 2. Toggle sidebar in Tab 1
# 3. Refresh Tab 2
# Expected: Tab 2 reflects the new state (localStorage shared across tabs)

# Test Case 6: Responsive Behavior Unchanged
# 1. Desktop: sidebar expanded, then resize to mobile
# Expected: Sidebar adapts to mobile defaults (current behavior)
# 2. Mobile: sidebar visible, then resize to desktop
# Expected: Sidebar adapts to desktop defaults (current behavior)
```

### Level 3: Browser Compatibility Testing
```bash
# Test in multiple browsers:
# - Chrome/Chromium (latest)
# - Firefox (latest)
# - Safari (latest)
# - Edge (latest)

# For each browser:
# 1. Verify toggle functionality works
# 2. Verify state persists across refresh
# 3. Check browser console for errors
# 4. Test in both desktop and mobile viewports

# Expected: Consistent behavior across all browsers
```

### Level 4: localStorage Inspection
```bash
# Browser DevTools > Application/Storage > Local Storage
# 1. Toggle sidebar
# 2. Inspect localStorage entry
# Expected:
#   Key: 'nextrek-sidebar-state'
#   Value: {"ariaExpanded":"false","dataMinimized":"false"} (or similar)

# 3. Manually edit localStorage value to invalid JSON
# 4. Refresh page
# Expected: Application handles gracefully, uses defaults

# 5. Clear localStorage
# 6. Refresh page
# Expected: Sidebar uses default responsive state
```

## Final Validation Checklist

- [ ] JavaScript syntax is valid (no console errors)
- [ ] Sidebar state persists across page refresh
- [ ] Sidebar state persists across Turbo navigation
- [ ] Desktop toggle works and saves state
- [ ] Mobile toggle works and saves state
- [ ] localStorage unavailable handled gracefully (private browsing)
- [ ] Responsive behavior unchanged (resize resets as before)
- [ ] Code includes clear, helpful comments
- [ ] No HTML structure changes
- [ ] No backend code changes
- [ ] Works in Chrome, Firefox, Safari, Edge
- [ ] localStorage entry has correct structure
- [ ] Corrupt localStorage data handled gracefully

---

## Anti-Patterns to Avoid

- ❌ Don't modify HTML structure or attributes - only JavaScript
- ❌ Don't change backend code - pure client-side solution
- ❌ Don't use sessionStorage - we need persistence across sessions
- ❌ Don't assume localStorage is always available - use feature detection
- ❌ Don't skip try-catch around JSON.parse/stringify - can throw errors
- ❌ Don't save state on every resize event - preserve current behavior
- ❌ Don't use cookies - localStorage is simpler and more appropriate
- ❌ Don't store complex objects - keep it simple (two attributes only)
- ❌ Don't break existing toggle functionality - add, don't replace
- ❌ Don't ignore mobile vs desktop state differences - handle both

---

## Expected Code Changes Summary

**File Modified**: `app/views/layouts/v3.html.erb`

**Lines Modified**: ~76-124 (script section)

**Changes**:
- Add 2 new functions (~20-30 lines)
- Modify 3 existing functions (~10 lines)
- Add comments (~15-20 lines)
- Total additions: ~50-60 lines of code
- Total modifications: ~10 lines

**No Changes**:
- HTML structure
- CSS/styling
- Backend code
- Other JavaScript files
- Database

---

## Reference Implementation Pattern

```javascript
// Modern JavaScript localStorage pattern (for reference)
const STORAGE_KEY = 'app-feature-state';

function getStoredState() {
  if (!window.localStorage) return null;

  try {
    const data = localStorage.getItem(STORAGE_KEY);
    return data ? JSON.parse(data) : null;
  } catch (e) {
    console.warn('Storage read error:', e);
    return null;
  }
}

function saveState(state) {
  if (!window.localStorage) return;

  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(state));
  } catch (e) {
    console.warn('Storage write error:', e);
  }
}

// Usage
const state = getStoredState();
if (state) {
  // Apply saved state
} else {
  // Use defaults
}

// After state change
saveState(newState);
```

---

**PRP Confidence Score**: 9/10

**Reasoning**:
- ✅ Complete context provided (existing code, patterns, structure)
- ✅ Clear, actionable tasks with specific locations
- ✅ Comprehensive validation strategy with test cases
- ✅ localStorage best practices documented
- ✅ Error handling patterns specified
- ✅ Browser compatibility addressed
- ✅ Anti-patterns clearly listed
- ✅ No external dependencies required
- ✅ Minimal code changes (low risk)
- ⚠️ Minor uncertainty: Turbo Rails cache behavior might require additional testing

**Success Probability**: Very High - This is a well-defined, isolated change with clear requirements and comprehensive validation. The AI should be able to implement this in one pass with the provided context.
