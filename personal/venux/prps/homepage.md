# Purpose

Build a pretty fashion and interactive homepage for the website

## Core Principles
1. **Progressive Success**: Start simple, validate, then enhance
2. **User-Centric Design**: Focus on user experience and interactivity
3. Research from existsing codebase and design patterns
4. Avoid to break other functions.

# What

The homepage includes following content and features:

- Randomly selected actress profiles at top of the page and present like welcome waifu with carousel-like way on the board. Each actress would have interactive behavior like chatting or conversation with user.
- Display latest works gallery along with actress profiles
- Display hottest works gallery along with actress profiles
- Display some photos of actress randomly selected from the database

# Success Criteria

- Pretty UI design of gallery website, like spotify, netflix or other modern web application
- The UI component are responsive and mobile-friendly
- Use Phoenix LiveView for dynamic, real-time user interfaces, and extract as components for reuseability. Like card, photo design.
- When showing image, use the lazy_img component predefined in the project to optimize loading performance.
- Interactive behavior of actress like AI actor, like chatting or conversation with user
- Efficiently display and performance of the Frontend codes, using tailwindcss and daisyui. The javascript and CSS assets are bundled with esbuild.


## List of task to be completed

1. Homepage Design
UPDATE homepage:
  - lib/venux_web/live/home_live/index.ex
  - lib/venux_web/live/home_live/index.html.heex

2. Extract Components
CREATE or UPDATE shared/reusable components in lib/venux_web/components:


3. Update Document
UPDATE README.md if necessary:
  - Add usage of new added components
  - Add doc for any pattern new designed

4. Update Memory
Store the memory of what you have done in the project memory
