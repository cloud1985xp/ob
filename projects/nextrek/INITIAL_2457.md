## FEATURE:

Implement Custom Budget Item Management

- To allow users to add custom budget items by year
- Implement the function at routes `/accountings/reports/show/budget/items
- A custom budget item is aimed to represent the custom accounting subject added to the yearly budget plan/report
- THe system already has in-built subject of budget plan/report, this feature will allow users to add their own custom subjects
  - Refer the default subjects full list from Accountings::BudgetReportPresenter::DEFAULT_SUBJECT_IDS
- When editing custom budget items,
  - Choose the year of the budget plan/report for editing, default to current year.
  - Load all subjects of the current_user's group, which are the subjects that can be used in the budget plan/report.
    - Called Group#cached_subjects to get the subjects of current_user's group.
  - In the form, it groups the subjects by the group of income statement sheets.
    - Call Accountings.group_by_statement_group(subjects) to get the subjects
  - In each group, user manage the budget items of the group.
    - List the items as a table with the following columns:
      - Subject Name
      - Subject Code
      - Type: (System default or added by user)
      - Action (delete button)
    - There are default subjects of the group, list them like item but not editable.
    - User is able to add more subject(budget itme) to the group
      - There is a button to add a new budget item at bottom of the group section
      - When clicked the button, it insert/appends a new row to the table with editable fields:
        - Subject (select input)
          - provide a select input to pick subject from the subjects pool of the group, except the default subjects.
      - Then save the added item
    - The budget item added to the group is allowed to be deleted
- Refer to the image file @ai_docs/budget_item_management.png for the UI design
  - Design the UI with responsive design in mind, using tailwindcss and daisyUI components
  - Make sure the UI is user-friendly and intuitive, modern and simple to use.
