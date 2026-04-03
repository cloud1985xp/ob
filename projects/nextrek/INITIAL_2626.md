## FEATURE:

Implement Admin Function to Manually Trigger Task


There are some task originally scheduled to run at specific intervals (e.g., daily, weekly). However, there are scenarios where an administrator may need to trigger these tasks manually outside of their regular schedule. This feature will allow admins to initiate these tasks on-demand through the admin interface.

Those tasks s are executed on ECS Fargate by running with Task Definition of

- nk-production-schedule-task (for production) or
- nk-staging-schedule-task (for staging)

You need to refer to config/schedule/production.yml or config/schedule/staging.yml for the related AWS resources info and configurations.

To invoke the task, you need to add aws-sdk-ecs gem to the Gemfile and use it the launch the ECS Fargate task execution.


I want to implement the following 2 functions in the admin interface:

1. A button to trigger ECS Fargate task of `rake export:order_data` to the page in /orders at admin console
2. A button to trigger ECS Fargate task of `rake export:group_report` to the page in /groups at admin console

### Key Requirements

- create a class to represent a Task for running, with a parameter of task_name which mapping to the rake task to be executed
  - the task name could also be the identifier (id) of the task, in order to use Redis::Objects to store some value in Redis for the task
  - when a task is triggered, store the task id and the timestamp of when it was triggered in Redis
  - each task can only have one running instance at a time, so stored the status in Redis to indicate whether the task is currently running or not
  - the object have ability to get task status (e.g., pending, running, completed, failed) by checking the ECS Fargate task status
  - the object have method to launch the ECS Fargate task execution using aws-sdk-ecs gem

- In the admin interface, add buttons to trigger the tasks on the respective pages
  - on /orders page, add a button "Export Order Data" to trigger the `rake export:order_data` task
  - on /groups page, add a button "Export Group Report" to trigger the `rake export:group_report` task
  - when the button is clicked, it should call the method in the Task class to launch the ECS Fargate task execution
  - display the current status of the task on the page (e.g., pending, running, completed, failed) and the timestamp of when it was last triggered

- Don't need to implement the rake tasks `rake export:order_data` and `rake export:group_report`, just the ability to trigger them and monitor their status. We assume the rake tasks are already implemented and available to be run on ECS Fargate.

### Implementation Steps

For orders as example:

- Add new action to the routes by POST to admin/orders/export, Admin::OrdersController#export action.
- in export action, create an instance of the Task class with task_name of `export:order_data`, call the method to launch the ECS Fargate task execution, and redirect back to the orders page with a flash message indicating the task has been triggered.
- in the view for /orders page,
  - add a button "Export Order Data" that submits a POST request to admin/orders/export path.
  - display the current status of the task and the timestamp of when it was last triggered by querying the Task class instance.

### Reference File you may need to modify

- app/controllers/admin/orders_controller
- app/controllers/admin/groups_controller
- app/views/admin/orders/index.html.erb
- app/views/admin/groups/index.html.erb


### Other Requirements

- Add the button to the index page by using existing styles and conventions in the admin interface. There already have some buttons at header of the page, just prepare the new button in similar way.
- Do not change existing functionality of the admin interface. The new feature should be added without affecting other parts of the system.

