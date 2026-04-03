# Custom Budget Item Management PRP

name: "Custom Budget Item Management Implementation"
description: |

## Purpose
Implement comprehensive custom budget item management functionality allowing users to add and remove custom accounting subjects to yearly budget plans, organized by statement groups with intuitive UI.

## Core Principles
1. **Context is King**: Leveraging existing budget infrastructure and patterns
2. **Validation Loops**: Rails-specific testing and linting validation
3. **Information Dense**: Using established codebase patterns and conventions  
4. **Progressive Success**: Start with core functionality, validate, then enhance UI
5. **Global rules**: Following all rules in CLAUDE.md

---

## Goal
Implement a complete budget item management interface at `/accountings/reports/show/budget/items` that allows users to:
- View default and custom budget items grouped by statement types
- Add custom accounting subjects as budget items for a specific year
- Remove custom budget items (but not default ones)
- Switch between different years to manage budget items

## Why
- **Business value**: Enables businesses to customize their budget reporting beyond default system subjects
- **Integration**: Seamlessly integrates with existing budget reporting and BudgetReportPresenter
- **User impact**: Provides intuitive management of budget line items for accurate financial planning

## What
A comprehensive budget item management system with:
- Year selection dropdown (defaults to current year)
- Statement group sections (営業収入, 営業成本, 営業費用, etc.)
- Tables showing Subject Name, Code, Type (System Default vs Custom), and Actions
- Add/remove functionality for custom items
- Responsive design using TailwindCSS and DaisyUI

### Success Criteria
- [ ] Users can view budget items organized by statement groups
- [ ] Users can add custom subjects as budget items
- [ ] Users can remove custom budget items
- [ ] Default items are displayed but not editable
- [ ] Year selection changes the data context
- [ ] UI matches provided mockup design
- [ ] All tests pass and code follows Ruby style guidelines

## All Needed Context

### Documentation & References
```yaml
# MUST READ - Include these in your context window
- file: app/presenters/accountings/budget_report_presenter.rb
  why: Contains DEFAULT_SUBJECT_IDS constant and custom_item_subjects method pattern
  lines: 11-12, 310-313

- file: app/models/accountings/budget_item.rb  
  why: Existing model with correct associations and for_year scope
  critical: Model already exists - don't recreate

- file: app/models/concerns/subject_cacheable.rb
  why: Provides Group#cached_subjects interface and by_statement_group method
  lines: 43-45, 122-123

- file: app/models/accountings.rb
  why: Contains group_by_statement_type method (NOT group_by_statement_group as mentioned in requirements)
  lines: 23-49
  critical: Method name is group_by_statement_type, not group_by_statement_group

- file: app/controllers/accountings/subjects_controller.rb
  why: Standard CRUD controller pattern to follow for budget_items_controller
  pattern: index, create, destroy actions with proper authorization

- file: app/controllers/accountings/base_controller.rb  
  why: Base class that handles common accountings functionality

- file: config/routes.rb
  why: Existing route structure at lines 318-319 for reports
  lines: 318-319

- file: ai_docs/budget_item_management.png
  why: UI mockup showing exact design requirements
  critical: Shows year selection, statement groups, table columns, add/delete buttons
```

### Current Codebase Patterns
```ruby
# CRITICAL: BudgetItem model already exists with proper associations
class Accountings::BudgetItem < ApplicationRecord
  belongs_to :group
  belongs_to :subject, class_name: 'Accountings::Subject'  
  scope :for_year, ->(year) { where(year: year) }
end

# CRITICAL: Use cached_subjects for performance
group.cached_subjects.by_statement_group  # groups by revenue/cost/expense types
group.cached_subjects.find_by_id(subject_ids)  # get subjects by IDs

# CRITICAL: Default subjects pattern from BudgetReportPresenter  
DEFAULT_SUBJECT_IDS = [4100, 4101, 4102, 5001, 5100, ...]
```

### Known Gotchas & Library Quirks
```ruby
# CRITICAL: Method name discrepancy in requirements
# Requirements mention: Accountings.group_by_statement_group(subjects)
# Actual method is: Accountings.group_by_statement_type(subjects, &:code)

# CRITICAL: Subject caching behavior
# Always use group.cached_subjects instead of group.accounting_subjects for performance
# Cache invalidation happens via group.clear_cached_subjects

# CRITICAL: Authorization pattern
# Use authorize_resource class: 'Accountings::BudgetItem' 
# OR authorize_resource class: 'Accountings::Budget' if managing budget permissions

# CRITICAL: Route integration
# Existing reports route: get '/reports/show/(/:category)(/:type)(/:date)' => 'reports#show'
# Need to add separate route for budget items management
```

## Implementation Blueprint

### Route Structure
Add to accountings namespace in routes.rb:
```ruby
namespace :accountings do
  # Add after existing routes around line 340
  get '/reports/show/budget/items' => 'budget_items#index', as: :budget_items_reports
  post '/reports/show/budget/items' => 'budget_items#create'  
  delete '/reports/show/budget/items/:id' => 'budget_items#destroy'
end
```

### List of tasks to be completed in order:

```yaml
Task 1: Create BudgetItemsController
CREATE app/controllers/accountings/budget_items_controller.rb:
  - INHERIT from: Accountings::BaseController
  - MIRROR authorization pattern from: app/controllers/accountings/subjects_controller.rb
  - IMPLEMENT actions: index, create, destroy
  - USE current_group and current_staff from base controller

Task 2: Update Routes Configuration  
MODIFY config/routes.rb:
  - FIND namespace :accountings around line 211
  - ADD budget items routes after existing reports route (line 319)
  - PRESERVE existing route structure

Task 3: Create Index View Template
CREATE app/views/accountings/budget_items/index.html.erb:
  - MIRROR layout patterns from: app/views/accountings/subjects/index.html.erb
  - IMPLEMENT year selection dropdown
  - CREATE statement group sections with tables
  - USE TailwindCSS and DaisyUI components as specified

Task 4: Create JavaScript Interactions
CREATE app/javascript/controllers/budget_items_controller.js:
  - FOLLOW Stimulus patterns from existing controllers
  - IMPLEMENT dynamic add/remove functionality  
  - HANDLE form submissions via fetch API
  - UPDATE UI state without full page reload

Task 5: Add Authorization Rules
MODIFY app/models/ability.rb (if exists) OR controller authorization:
  - ADD budget item management permissions
  - LINK to existing budget permissions or create new resource
  - FOLLOW patterns from other accounting resources

Task 6: Create Comprehensive Tests
CREATE spec/controllers/accountings/budget_items_controller_spec.rb:
  - MIRROR test patterns from: spec/controllers/accountings/subjects_controller_spec.rb  
  - TEST all CRUD operations
  - TEST authorization scenarios
  - TEST year filtering functionality

Task 7: Add Feature Tests
CREATE spec/features/budget_items_spec.rb:
  - TEST complete user workflow
  - VERIFY UI interactions work correctly
  - TEST JavaScript functionality
```

### Controller Implementation Details
```ruby
# app/controllers/accountings/budget_items_controller.rb
module Accountings
  class BudgetItemsController < Accountings::BaseController
    authorize_resource class: 'Accountings::BudgetItem'
    
    def index
      @year = params[:year]&.to_i || Date.current.year
      @statement_groups = build_statement_groups
    end
    
    def create
      # PATTERN: Simple create without form object (like subjects_controller)
      budget_item = current_group.accounting_budget_items.new(budget_item_params)
      
      if budget_item.save
        # PATTERN: AJAX response for dynamic UI update
        render json: { status: 'success', item: serialize_budget_item(budget_item) }
      else
        render json: { status: 'error', errors: budget_item.errors.full_messages }
      end
    end
    
    def destroy
      # PATTERN: Find and destroy with proper error handling
      budget_item = current_group.accounting_budget_items.find(params[:id])
      budget_item.destroy
      render json: { status: 'success' }
    end
    
    private
    
    def build_statement_groups
      # CRITICAL: Use group_by_statement_type (not group_by_statement_group)
      statement_groups = current_group.cached_subjects.by_statement_group
      
      statement_groups.map do |type, subjects|
        {
          type: type,
          name: I18n.t(type, scope: 'accountings.reports.budget_report.statement_group'),
          default_subjects: get_default_subjects_for_type(subjects),
          custom_subjects: get_custom_subjects_for_type(subjects)
        }
      end
    end
end
```

### UI Structure Pattern
```erb
<!-- PATTERN: Follow existing accountings views structure -->
<div class="bg-white rounded-lg shadow">
  <div class="p-6">
    <h1>會計項目管理</h1>
    
    <!-- Year selection dropdown -->
    <%= form_with url: accountings_budget_items_reports_path, method: :get, class: "mb-6" do |f| %>
      <%= f.select :year, year_options, { selected: @year }, 
                   { class: "select select-bordered", 
                     onchange: "this.form.submit()" } %>
    <% end %>
    
    <!-- Statement groups -->
    <% @statement_groups.each do |group| %>
      <div class="mb-8" data-controller="budget-items" data-budget-items-type-value="<%= group[:type] %>">
        <h3><%= group[:name] %> (<%= group[:default_subjects].size + group[:custom_subjects].size %> 項目)</h3>
        
        <!-- Table with subjects -->
        <table class="table table-zebra w-full">
          <!-- Default subjects (not editable) -->
          <!-- Custom subjects (with delete buttons) -->
        </table>
        
        <button class="btn btn-primary" data-action="click->budget-items#addItem">
          新增預算項目
        </button>
      </div>
    <% end %>
  </div>
</div>
```

## Validation Loop

### Level 1: Syntax & Style  
```bash
# Run these FIRST - fix any errors before proceeding
bundle exec rubocop app/controllers/accountings/budget_items_controller.rb --auto-correct
bundle exec rubocop app/views/accountings/budget_items/ --auto-correct

# Expected: No rubocop violations. Fix any style issues.
```

### Level 2: Unit Tests
```ruby
# spec/controllers/accountings/budget_items_controller_spec.rb
RSpec.describe Accountings::BudgetItemsController do
  let(:group) { create(:group) }
  let(:staff) { create(:staff, group: group) }
  let(:subject) { create(:accountings_subject, group: group) }
  
  describe 'GET #index' do
    it 'loads statement groups correctly' do
      get :index
      expect(assigns(:statement_groups)).to be_present
      expect(response).to be_successful
    end
  end
  
  describe 'POST #create' do
    it 'creates budget item successfully' do
      expect {
        post :create, params: { 
          budget_item: { accountings_subject_id: subject.id, year: 2024 }
        }
      }.to change(Accountings::BudgetItem, :count).by(1)
    end
  end
  
  describe 'DELETE #destroy' do
    it 'removes budget item' do
      budget_item = create(:accountings_budget_item, group: group)
      expect {
        delete :destroy, params: { id: budget_item.id }
      }.to change(Accountings::BudgetItem, :count).by(-1)
    end
  end
end
```

```bash
# Run and iterate until passing:
bundle exec rspec spec/controllers/accountings/budget_items_controller_spec.rb -v
# If failing: Read error, fix code, re-run
```

### Level 3: Feature Test
```ruby  
# spec/features/budget_items_spec.rb
RSpec.describe 'Budget Items Management', type: :feature, js: true do
  it 'allows adding and removing custom budget items' do
    visit accountings_budget_items_reports_path
    
    # Test year selection
    select '2024', from: 'year'
    expect(page).to have_current_path(/year=2024/)
    
    # Test adding item
    within first('.statement-group') do
      click_button '新增預算項目'
      select subject.name, from: 'subject_select'
      click_button '儲存'
      
      expect(page).to have_content(subject.name)
      expect(page).to have_content('自訂項目')
    end
    
    # Test removing item  
    within first('.custom-item') do
      click_button '刪除'
      expect(page).not_to have_content(subject.name)
    end
  end
end
```

```bash
# Run integration test:
bundle exec rspec spec/features/budget_items_spec.rb -v
# Expected: All scenarios pass with JS interactions working
```

### Level 4: Manual Integration Test
```bash
# Start Rails server
bundle exec rails server

# Navigate to: http://localhost:3000/accountings/reports/show/budget/items
# Test:
# 1. Year selection dropdown changes data
# 2. Default items show as "系統預設" and cannot be deleted  
# 3. Add custom items using dropdown
# 4. Delete custom items using delete button
# 5. Check browser console for JS errors

# Expected: Full functionality working as per UI mockup
```

## Final Validation Checklist
- [ ] All tests pass: `bundle exec rspec spec/controllers/accountings/budget_items_controller_spec.rb`
- [ ] No linting errors: `bundle exec rubocop app/controllers/accountings/budget_items_controller.rb`
- [ ] Feature tests pass: `bundle exec rspec spec/features/budget_items_spec.rb`
- [ ] Manual test successful: Navigate to budget items page and test CRUD operations
- [ ] Year selection works correctly
- [ ] Statement groups display properly
- [ ] Default vs custom items are distinguished  
- [ ] JavaScript interactions work smoothly
- [ ] UI matches provided mockup design
- [ ] Authorization prevents unauthorized access

---

## Anti-Patterns to Avoid
- ❌ Don't create new BudgetItem model - it already exists
- ❌ Don't use `group_by_statement_group` - method is `group_by_statement_type`  
- ❌ Don't bypass cached_subjects - use for performance
- ❌ Don't skip authorization checks
- ❌ Don't hardcode subject IDs - use DEFAULT_SUBJECT_IDS constant
- ❌ Don't ignore the UI mockup design requirements
- ❌ Don't create complex form objects for simple CRUD operations

---

**Implementation Confidence Score: 8/10**

High confidence due to:
- Clear existing patterns and infrastructure
- Detailed UI mockup provided
- Well-established codebase conventions
- Existing BudgetItem model with correct associations

Main risks: JavaScript UI interactions and proper subject filtering logic.