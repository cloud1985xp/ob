## Improve Javascript in Layout v3 Template

Currently there is a layout call v3 at app/views/layouts/v3.html.erb

In the layout HTML there is a script to toggle Sidebar Menu to expand and collapse.

But it's not a good situation to place javascript directly in the layout .erb file

Extract and refactor those scripts to separated file then import and use it from the main js entry point at app/javascript/v3.js

It requires to:
- Extract the scripts to the file at app/javascript/components/sidebar_menu.js with modulize design to use it in esbuild way
- Import and apply the module in app/javascript/v3.js to use the sidebar menu component.
- The sidebar component should be work as same behavior as original
- Consider the whole application has using hotwire/turbo, make sure the sidebar menu works well when navigating pages with turbo.
- Refactor the code to modern and clean coding style

