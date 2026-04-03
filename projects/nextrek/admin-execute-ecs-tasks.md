# PRP: Admin Manual ECS Fargate Task Execution

name: "Admin Manual ECS Fargate Task Execution"
description: |
  Enable administrators to manually trigger scheduled ECS Fargate tasks (export:order_data and export:group_report)
  through the admin interface with status tracking and concurrent execution prevention.

---

## Goal
Implement admin functionality to manually trigger ECS Fargate tasks that are normally scheduled, with real-time status tracking and prevention of concurrent executions.

## Why
- **Operational Flexibility**: Admins need to trigger data exports outside regular schedules for urgent requests or troubleshooting
- **User Impact**: Reduces response time for custom data export requests from hours/days to minutes
- **Integration**: Leverages existing ECS Fargate infrastructure and scheduled task definitions
- **Problems Solved**: Eliminates manual SSH access or developer intervention for running export tasks

## What
Two admin interface buttons to manually trigger ECS Fargate tasks with status display:
1. "Export Order Data" button on `/admin/orders` page → triggers `rake export:order_data`
2. "Export Group Report" button on `/admin/groups` page → triggers `rake export:group_report`

### Success Criteria
- [ ] Admin can click button to trigger ECS task execution
- [ ] Task status displays on page (pending, running, completed, failed)
- [ ] Last triggered timestamp is visible
- [ ] Only one instance of each task can run at a time
- [ ] Tasks run on ECS Fargate with correct network configuration
- [ ] All tests pass and no linting errors
- [ ] No security vulnerabilities introduced

---

## All Needed Context

### Documentation & References
```yaml
# AWS SDK for Ruby - ECS Client
- url: https://docs.aws.amazon.com/sdk-for-ruby/v3/api/Aws/ECS/Client.html
  why: Main documentation for ECS client methods
  critical: run_task and describe_tasks methods

- url: https://docs.aws.amazon.com/sdk-for-ruby/v2/api/Aws/ECS/Types/RunTaskRequest.html
  why: Detailed parameters for run_task including Fargate network configuration
  critical: network_configuration with awsvpc_configuration is REQUIRED for Fargate

- url: https://stackoverflow.com/questions/67607006/how-to-configure-ephemeral-storage-on-ecs-fargate-task-via-ruby-sdk
  why: Real-world example of run_task with Fargate network configuration
  critical: Shows complete run_task call with all required params

# Existing Codebase Patterns
- file: app/models/init_progress.rb
  why: Reference pattern for Redis::Objects usage (value, set, id construction)
  critical: ID format is "#{@object.class.name.underscore}:#{@object.id}"

- file: app/models/performance.rb
  why: Another Redis::Objects pattern with AASM state machine
  critical: Shows how to use Redis value and set for state tracking

- file: config/schedule/deploy.rb
  why: AWS ECS client initialization and task definition patterns
  critical: Shows Aws::ECS::Client.new(region:) usage and task ARN construction

- file: app/controllers/admin/orders_controller.rb (lines 48-59)
  why: Existing export action pattern with background job
  critical: Use standard_flash_message and redirect pattern

- file: app/controllers/admin/groups_controller.rb (lines 29-55)
  why: Another export action with multiple export types
  critical: Shows authorization checks and export parameter handling

- file: app/views/admin/orders/index.html.erb (lines 10-17)
  why: Button styling pattern for admin interface
  critical: Use "page-navi btn-group" with Font Awesome icons

- file: app/views/admin/groups/index.html.erb (lines 15-31)
  why: Dropdown button pattern for multiple export options
  critical: Shows Bootstrap 5 dropdown pattern with btn-outline classes

- file: config/schedule/production.yml
  why: AWS ECS configuration including cluster, subnets, security groups
  critical: Network configuration required for Fargate tasks

- file: config/schedule/staging.yml
  why: Staging environment AWS configuration
  critical: Different subnets/security groups per environment
```

### Current Codebase Structure
```bash
app/
├── controllers/
│   └── admin/
│       ├── orders_controller.rb      # Has export action (line 48-59)
│       └── groups_controller.rb      # Has export action (line 29-55)
├── models/
│   ├── init_progress.rb              # Redis::Objects pattern
│   └── performance.rb                # Redis::Objects with state machine
├── views/
│   └── admin/
│       ├── orders/
│       │   └── index.html.erb        # Button pattern (line 10-17)
│       └── groups/
│           └── index.html.erb        # Dropdown pattern (line 15-31)
config/
├── routes.rb                          # Admin routes (line 56-74)
└── schedule/
    ├── production.yml                 # AWS config for production
    └── staging.yml                    # AWS config for staging
Gemfile                                # Already has aws-sdk-s3, aws-sdk-ec2
```

### Desired Codebase Structure
```bash
app/
├── services/
│   └── admin/
│       └── ecs_task.rb               # NEW: ECS task wrapper with Redis tracking
├── controllers/
│   └── admin/
│       ├── orders_controller.rb      # MODIFY: Add export_data action
│       └── groups_controller.rb      # MODIFY: Add export_report action
├── views/
│   └── admin/
│       ├── orders/
│       │   └── index.html.erb        # MODIFY: Add Export button + status
│       └── groups/
│           └── index.html.erb        # MODIFY: Add Export button + status
config/
└── routes.rb                          # MODIFY: Add POST routes
Gemfile                                # MODIFY: Add aws-sdk-ecs gem
```

### Known Gotchas & Library Quirks
```ruby
# CRITICAL: aws-sdk-ecs gem MUST be added to Gemfile
# Add: gem 'aws-sdk-ecs'

# CRITICAL: Fargate tasks REQUIRE network_configuration
# Missing this will cause "InvalidParameterException"
network_configuration: {
  awsvpc_configuration: {
    subnets: [...],                    # From config/schedule/{env}.yml
    security_groups: [...],             # From config/schedule/{env}.yml
    assign_public_ip: 'ENABLED'        # Required for internet access
  }
}

# CRITICAL: Task definition names follow this pattern
# Production: "nk-production-schedule-task-weekly-order-report"
# Staging: "nk-staging-schedule-task-weekly-order-report"
# The task definitions already exist from scheduled tasks

# CRITICAL: ECS run_task returns task ARN in format:
# "arn:aws:ecs:region:account:task/cluster-name/task-id"
# Use this ARN with describe_tasks to check status

# CRITICAL: Redis::Objects ID must be unique per task
# Use task_name as ID: "export:order_data", "export:group_report"
# This allows direct lookup without needing database records

# GOTCHA: AWS credentials come from ENV variables (see existing S3 usage)
# No explicit credentials needed in client initialization

# GOTCHA: Only ONE instance of each task should run at a time
# Use Redis value to store current running status
# Check status before allowing new execution

# PATTERN: Rails.env determines which config file to use
# production.yml for production, staging.yml for staging
```

---

## Implementation Blueprint

### Data Models & Structure
```ruby
# app/services/admin/ecs_task.rb
# This class wraps ECS Fargate task execution with Redis-based state tracking

class Admin::EcsTask
  include Redis::Objects

  # Redis storage
  value :status              # Current status: pending, running, completed, failed
  value :task_arn           # ECS task ARN for the running task
  value :triggered_at       # Timestamp of last trigger
  value :error_message      # Error details if failed

  attr_reader :task_name

  # Initialize with task name (e.g., 'export:order_data')
  def initialize(task_name)
    @task_name = task_name
  end

  # Redis::Objects requires id method
  def id
    task_name  # Use task_name directly as ID
  end

  # Launch the ECS Fargate task
  def execute!
    # Check if already running
    # Get AWS config for current environment
    # Build run_task parameters
    # Execute via AWS ECS client
    # Store task ARN and update status
  end

  # Check current task status from ECS
  def refresh_status!
    # If no task_arn, return current status
    # Call describe_tasks with task_arn
    # Update status based on ECS response
    # Clear task_arn if task completed/stopped
  end

  # Check if task can be executed
  def can_execute?
    status.value != 'running'
  end

  private

  def ecs_client
    # Initialize Aws::ECS::Client with region
  end

  def aws_config
    # Load from config/schedule/#{Rails.env}.yml
  end

  def task_definition_name
    # Map task_name to ECS task definition
    # 'export:order_data' -> 'nk-{env}-schedule-task-weekly-order-report'
    # 'export:group_report' -> 'nk-{env}-schedule-task-weekly-group-report'
  end

  def run_task_params
    # Build complete run_task parameters hash
  end
end
```

---

## Implementation Tasks

### Task 1: Add aws-sdk-ecs gem to Gemfile
```yaml
MODIFY Gemfile:
  - FIND: gem 'aws-sdk-s3'
  - INSERT AFTER: gem 'aws-sdk-ecs'
  - RUN: bundle install
```

### Task 2: Create Admin::EcsTask service class
```yaml
CREATE app/services/admin/ecs_task.rb:
  - INCLUDE Redis::Objects for state tracking
  - IMPLEMENT initialize(task_name) method
  - IMPLEMENT id method returning task_name
  - IMPLEMENT execute! method to launch ECS task
  - IMPLEMENT refresh_status! method to check task status
  - IMPLEMENT can_execute? method to prevent concurrent runs
  - IMPLEMENT private helper methods:
    * ecs_client: Initialize Aws::ECS::Client
    * aws_config: Load YAML config for current environment
    * task_definition_name: Map task_name to ECS definition
    * run_task_params: Build complete parameter hash
  - PATTERN: Follow Redis::Objects pattern from init_progress.rb
  - CRITICAL: Use task_name as Redis key ID
```

**Pseudocode for execute! method:**
```ruby
def execute!
  # Validation
  raise "Task already running" unless can_execute?

  # Update status
  self.status.value = 'running'
  self.triggered_at.value = Time.current.iso8601

  begin
    # Get AWS configuration
    config = aws_config

    # Launch ECS task
    response = ecs_client.run_task(run_task_params)

    # Store task ARN from response
    task = response.tasks.first
    self.task_arn.value = task.task_arn

    # Return success
    { success: true, task_arn: task.task_arn }
  rescue Aws::ECS::Errors::ServiceError => e
    # Handle AWS errors
    self.status.value = 'failed'
    self.error_message.value = e.message
    { success: false, error: e.message }
  end
end
```

**Pseudocode for run_task_params:**
```ruby
def run_task_params
  config = aws_config

  {
    cluster: config['aws']['cluster_name'],
    task_definition: task_definition_name,
    launch_type: 'FARGATE',
    network_configuration: {
      awsvpc_configuration: {
        subnets: config['aws']['fargate_subnet_ids'],
        security_groups: [config['aws']['fargate_security_group_ids']],
        assign_public_ip: 'ENABLED'
      }
    },
    overrides: {
      container_overrides: [{
        name: task_container_name,  # Extract from task_name
        command: task_command        # ['rake', 'export:order_data']
      }]
    }
  }
end
```

### Task 3: Add export_data action to Admin::OrdersController
```yaml
MODIFY app/controllers/admin/orders_controller.rb:
  - ADD new action export_data (after update method)
  - IMPLEMENT authorization check (management? or similar)
  - CREATE EcsTask instance with 'export:order_data'
  - CALL execute! method
  - USE standard_flash_message for success/failure
  - REDIRECT to admin_orders_path
  - PATTERN: Follow existing export method at lines 55-59
```

**Pseudocode:**
```ruby
def export_data
  # Authorization
  raise CanCan::AccessDenied unless current_admin_user.management?

  # Create task instance
  task = Admin::EcsTask.new('export:order_data')

  # Execute
  result = task.execute!

  # Flash message
  if result[:success]
    standard_flash_message(true, "訂單資料匯出任務已啟動")
  else
    standard_flash_message(false, "啟動失敗: #{result[:error]}")
  end

  # Redirect
  redirect_to admin_orders_path
end
```

### Task 4: Add export_report action to Admin::GroupsController
```yaml
MODIFY app/controllers/admin/groups_controller.rb:
  - ADD new action export_report (after export method)
  - IMPLEMENT authorization check
  - CREATE EcsTask instance with 'export:group_report'
  - CALL execute! method
  - USE standard_flash_message
  - REDIRECT to admin_groups_path
  - PATTERN: Similar to export_data but for groups
```

### Task 5: Add routes for new actions
```yaml
MODIFY config/routes.rb:
  - FIND: resources :orders line (around line 30)
  - ADD: post 'export_data', on: :collection

  - FIND: resources :groups do block (around line 56)
  - ADD: post 'export_report', on: :collection

  - PATTERN: Follow existing post :export pattern at line 66
```

### Task 6: Add Export button to orders index view
```yaml
MODIFY app/views/admin/orders/index.html.erb:
  - FIND: <div class="col-lg-4"> section with page-navi (lines 9-18)
  - ADD new button AFTER existing buttons:
    * TEXT: "匯出訂單資料 (手動執行)"
    * URL: export_data_admin_orders_path
    * METHOD: POST
    * CLASS: btn btn-warning (different color to distinguish)
    * ICON: fas fa-tasks
    * DATA: { confirm: '確定要執行訂單資料匯出任務？' }

  - ADD status display BELOW buttons:
    * CREATE task instance in controller index action
    * CALL refresh_status! to get current status
    * DISPLAY status badge and timestamp if available
    * STYLE: Use Bootstrap badges (badge bg-success/warning/danger)

  - PATTERN: Follow existing button pattern at lines 11-16
```

**View code example:**
```erb
<%= link_to export_data_admin_orders_path, method: :post,
    class: 'btn btn-warning',
    style: 'margin-right: 5px;',
    data: { confirm: '確定要執行訂單資料匯出任務？' } do %>
  <i class="fas fa-tasks"></i> 匯出訂單資料 (手動執行)
<% end %>

<!-- Status display -->
<% task = Admin::EcsTask.new('export:order_data') %>
<% task.refresh_status! %>
<% if task.status.value.present? %>
  <div class="mt-2">
    狀態: <span class="badge bg-<%= status_color(task.status.value) %>">
      <%= task.status.value %>
    </span>
    <% if task.triggered_at.value.present? %>
      | 最後執行: <%= Time.parse(task.triggered_at.value).in_time_zone.strftime('%Y-%m-%d %H:%M') %>
    <% end %>
  </div>
<% end %>
```

### Task 7: Add Export button to groups index view
```yaml
MODIFY app/views/admin/groups/index.html.erb:
  - FIND: admin-group-csv section with dropdown (lines 15-31)
  - ADD new standalone button (NOT in dropdown):
    * TEXT: "匯出群組報表 (手動執行)"
    * URL: export_report_admin_groups_path
    * METHOD: POST
    * CLASS: btn btn-warning btn-outline
    * ICON: fas fa-tasks
    * DATA: { confirm: '確定要執行群組報表匯出任務？' }

  - ADD status display similar to orders
  - PATTERN: Follow dropdown pattern but create standalone button
```

### Task 8: Add helper method for status color
```yaml
MODIFY app/controllers/admin/orders_controller.rb:
  - ADD helper_method :task_status_color
  - IMPLEMENT method to return Bootstrap color class

MODIFY app/controllers/admin/groups_controller.rb:
  - ADD same helper_method
```

**Pseudocode:**
```ruby
def task_status_color(status)
  case status
  when 'running' then 'primary'
  when 'completed' then 'success'
  when 'failed' then 'danger'
  else 'secondary'
  end
end
helper_method :task_status_color
```

---

## Validation Loop

### Level 1: Syntax & Style
```bash
# Run RuboCop on all modified files
bundle exec rubocop app/services/admin/ecs_task.rb --fix
bundle exec rubocop app/controllers/admin/orders_controller.rb --fix
bundle exec rubocop app/controllers/admin/groups_controller.rb --fix

# Expected: No offenses detected
# If errors: READ the offense description and FIX the code
```

### Level 2: Security Scan
```bash
# Run Brakeman security scanner
bundle exec brakeman

# Expected: No new security warnings
# If warnings: Review and address each one
# CRITICAL: Ensure no mass assignment vulnerabilities
# CRITICAL: Ensure proper authorization on new actions
```

### Level 3: Unit Tests
```ruby
# CREATE spec/services/admin/ecs_task_spec.rb

require 'rails_helper'

RSpec.describe Admin::EcsTask do
  let(:task_name) { 'export:order_data' }
  let(:task) { described_class.new(task_name) }

  before do
    # Clear Redis state before each test
    task.status.clear
    task.task_arn.clear
    task.triggered_at.clear
  end

  describe '#initialize' do
    it 'sets task_name' do
      expect(task.task_name).to eq(task_name)
    end
  end

  describe '#id' do
    it 'returns task_name as id' do
      expect(task.id).to eq(task_name)
    end
  end

  describe '#can_execute?' do
    context 'when no task is running' do
      it 'returns true' do
        expect(task.can_execute?).to be true
      end
    end

    context 'when task is running' do
      before { task.status.value = 'running' }

      it 'returns false' do
        expect(task.can_execute?).to be false
      end
    end
  end

  describe '#execute!' do
    let(:ecs_client) { instance_double(Aws::ECS::Client) }
    let(:task_response) do
      double(tasks: [double(task_arn: 'arn:aws:ecs:region:account:task/cluster/id')])
    end

    before do
      allow(Aws::ECS::Client).to receive(:new).and_return(ecs_client)
      allow(ecs_client).to receive(:run_task).and_return(task_response)
    end

    it 'launches ECS task successfully' do
      result = task.execute!
      expect(result[:success]).to be true
      expect(task.status.value).to eq('running')
    end

    it 'stores task ARN' do
      task.execute!
      expect(task.task_arn.value).to be_present
    end

    it 'stores triggered timestamp' do
      task.execute!
      expect(task.triggered_at.value).to be_present
    end

    context 'when AWS raises error' do
      before do
        allow(ecs_client).to receive(:run_task).and_raise(
          Aws::ECS::Errors::ServiceError.new(nil, 'Test error')
        )
      end

      it 'returns failure result' do
        result = task.execute!
        expect(result[:success]).to be false
      end

      it 'sets status to failed' do
        task.execute!
        expect(task.status.value).to eq('failed')
      end
    end
  end
end
```

```bash
# Run tests
bundle exec rspec spec/services/admin/ecs_task_spec.rb -v

# Expected: All tests passing
# If failing: READ the error message, understand root cause, FIX code, re-run
```

### Level 4: Manual Integration Test
```bash
# 1. Start Rails server
bundle exec rails server

# 2. Login to admin console
# Navigate to: http://admin.localhost:3000/orders

# 3. Click "匯出訂單資料 (手動執行)" button
# Expected:
#   - Flash message: "訂單資料匯出任務已啟動"
#   - Status shows: "running"
#   - Timestamp displays current time

# 4. Check AWS ECS Console
# Navigate to ECS > Clusters > nk-{env}-worker > Tasks
# Expected: See new task with status "RUNNING"

# 5. Check CloudWatch Logs
# Navigate to CloudWatch > Log groups > nk-{env}-schedule-task
# Expected: See log stream for the task execution

# 6. Wait for task completion (or refresh status)
# Expected: Status updates to "completed" or "failed"

# 7. Repeat for groups export
# Navigate to: http://admin.localhost:3000/groups
# Click "匯出群組報表 (手動執行)"
# Verify similar behavior
```

---

## Final Validation Checklist
- [ ] Gemfile includes aws-sdk-ecs gem
- [ ] Bundle install completed successfully
- [ ] Admin::EcsTask class created with Redis::Objects
- [ ] Controllers have new export actions with authorization
- [ ] Routes added for POST to export endpoints
- [ ] Views have buttons with Font Awesome icons and Bootstrap styling
- [ ] Status display shows current task state and timestamp
- [ ] All RuboCop offenses resolved
- [ ] No new Brakeman security warnings
- [ ] All unit tests pass
- [ ] Manual test successfully launches ECS task
- [ ] Task appears in AWS ECS console
- [ ] Status updates correctly after task completion
- [ ] Concurrent execution prevented (clicking twice doesn't launch two tasks)
- [ ] Error messages display clearly if task fails

---

## Anti-Patterns to Avoid
- ❌ Don't create ActiveRecord model for EcsTask (use Redis::Objects only)
- ❌ Don't skip authorization checks on export actions
- ❌ Don't hardcode AWS region/cluster (load from config files)
- ❌ Don't allow multiple concurrent executions of same task
- ❌ Don't forget network_configuration for Fargate launch_type
- ❌ Don't use assign_public_ip: 'DISABLED' (tasks need internet access)
- ❌ Don't create new button styling patterns (follow existing patterns)
- ❌ Don't skip i18n for Chinese text (use existing pattern of inline Chinese)
- ❌ Don't expose task execution to non-management admin users
- ❌ Don't catch generic Exception (catch specific Aws::ECS::Errors)

---

## PRP Confidence Score: 8/10

**Strengths:**
- Complete context with all necessary file references
- Executable validation gates at each level
- Real code examples from the codebase
- AWS SDK documentation URLs with specific sections
- Clear task breakdown with pseudocode
- Network configuration details from actual config files

**Risks:**
- Task definition names may need adjustment (assumed based on schedule config)
- Container names for overrides may need verification
- Environment-specific config loading might need Rails.env handling
- Redis::Objects may need additional configuration not visible in examples

**Mitigation:**
- Validation loop includes manual AWS console verification
- Error handling for AWS API failures
- Clear documentation of assumptions for AI to verify

The PRP provides sufficient context for one-pass implementation with iterative refinement through the validation loops.
