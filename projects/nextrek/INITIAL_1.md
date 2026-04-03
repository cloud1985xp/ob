## Improve Sidebar Menu in v3 Layout

Currently there is a layout call v3 at app/views/layouts/v3.html.erb

In the layout HTML there is a script to toggle Sidebar Menu to expand and collapse.

Please improve the script to have ability to remember the last state (expanded or collapsed) of the Sidebar Menu when user refresh the page or navigate to another page.

It requires to:

- Remember the last state (expanded or collapsed) of the Sidebar Menu using browser's local storage or cookies.
- Keep origin state when user refresh the page or navigate to another page.
- Keep origin behavior of toggling Sidebar Menu when user clicks on the toggle button. You can try to improve the code quality
- Do not change any backend code or HTML structure, only modify the script part in the layout HTML file.
- Addd comment to the script code to explain the logic if necessary
- Make sure the script works in all modern browsers. The coding style should be modern and clean.
