---
title: "Abusing LiveView's new Async Assigns Feature"
source: "https://fly.io/phoenix-files/abusing-liveview-new-async-assigns-feature/"
author:
published: 2023-11-09
created: 2026-04-07
description: "Documentation and guides from the team at Fly.io."
tags:
  - "clippings"
---
![Night sky with glowing moon and alien flying saucer swooping around the sky grabbing stars.](https://fly.io/phoenix-files/abusing-liveview-new-async-assigns-feature/assets/abusing-liveview-async-cover.webp)

Annie Ruygt

We’re Fly.io. We run apps for our users on hardware we host around the world. Fly.io happens to be a great place to run Phoenix applications. Check out how to [get started](https://fly.io/docs/elixir/)!

Previously, in [Star-Crossed LiveView Processes](https://fly.io/phoenix-files/star-cross-live-view-processes/), we explored using Elixir’s linked processes to connect a [Task](https://hexdocs.pm/elixir/Task.html) and a [LiveView](https://hexdocs.pm/phoenix_live_view/welcome.html) together. This required doing some extra work like trapping exits and handling received EXIT messages, but, in the end it work just like we wanted.

Here, we’ll revisit solving the same problem but using Phoenix LiveView’s newly built-in [async operations](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-async-operations) and see how it compares. [Chris McCord talked about this in his ElixirConf 2023 keynote video.](https://youtu.be/Ckgl9KO4E4M?si=8dttj5HEHaebZX6z&t=2112)

Before we go further, just know you can grab the finished code for this article in [this gist](https://gist.github.com/brainlid/e56a1e04aa8babd8bc6b851d5a07eda5).

We’ll briefly review the problem we are solving that was tackled in [Star-Crossed LiveView Processes](https://fly.io/phoenix-files/star-cross-live-view-processes/). If you’re already clear on what we’re doing here, jump ahead to [Starting the Task](#starting-the-task).

## Recap: What problem are we solving?

We want our LiveView to launch an asynchronous process. With LiveView v0.20, some incredible new [async operation](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-async-operations) features were added. This provides new building blocks for our LiveView to execute work in a separate but linked process.

The primary use case for these **async operations** is [described in the docs](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-async-operations) this way:

> It allows the user to get a working UI quickly while the system fetches some data in the background or talks to an external service, without blocking the render or event handling. For async work, you also typically need to handle the different states of the async operation, such as loading, error, and the successful result. You also want to catch any errors or exits and translate it to a meaningful update in the UI rather than crashing the user experience.

This works really well for most use cases. Think of loading a LiveView then starting a separate process to go fetch some data that will be immediately displayed once it’s ready.

In our use case, we are deviating from the happy path. From the docs, note the focus is on wanting the final “successful result”. Here’s how our use case differs:

- We’re not starting the process when the LiveView mounts. It happens as the user interacts with the LiveView.
- Rather than wait for the final result, we want a real-time stream of messages.

Here’s a visualization for the focus being more on the messages sent back while the process is running rather than the final result.

![Process overview of creating a task and receiving messages during processing versus getting the result only at the end.](https://fly.io/phoenix-files/abusing-liveview-new-async-assigns-feature/assets/async-task-overview-and-versus.png?2/3&centered)

Before we jump into the code, let’s define a bit more about how we want our application to behave.

## Goals for application behavior

Let’s outline how the application should behave, particularly around the async process and our LiveView.

- **The LiveView process remains unblocked.** Our blocking external calls should happen in a separate process. Let’s keep the UI buttery smooth.
- **When the LiveView process goes away, the other process should too.** Because the worker process only exists to fetch and provide data to the LiveView, we want the async process to stop if the user closes the page or navigates away.
- **When the async process crashes, it should NOT kill our LiveView.** We expect an async process talking to an external API will fail sometimes. We do not want that to crash the UI for the user.
- **Ability to cancel a running worker process.** We want the ability to cancel a running async worker from the LiveView.
- **We care about the side-effects, not the final result.** Think of a ChatGPT-like text steam, that’s what we want too. We prefer to show the user what we have now rather than hiding received data until it’s fully complete. These small chunks of data are still meaningful for the user and are the side-effects we’re interested in.

All right! That may seem like a hefty set of requirements, but it gives us the user experience we want and now using **async assigns** makes it even easier that it was before!

Here is a visual example of the behavior we want.

![Animated GIF showing the desired UX for starting and stopping a Task in a LiveView.](https://fly.io/phoenix-files/abusing-liveview-new-async-assigns-feature/assets/task-test-output-ux.gif?card&centered)

With an idea of what we’re building, let’s dive in and see how we start the async task!

## Starting the task

In this article we’re more concerned with *how* the LiveView and async task interact and we’re not focusing on the actual work the async operation is doing.

When the user clicks the “Start” button, the `handle_event` clears the messages and starts the Task.

```elixir
def handle_event("start", _params, socket) do
  socket =
    socket
    |> assign(:messages, [])
    |> start_test_task()

  {:noreply, socket}
end
```

In our `start_test_task` function, we use the new [`start_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#start_async/3) which takes an anonymous function that is executed asynchronously.

```elixir
def start_test_task(socket) do
  live_view_pid = self()

  socket
  |> assign(:async_result, AsyncResult.loading())
  |> start_async(:running_task, fn ->
    # the code to run async
    Enum.each(1..5, fn n ->
      Process.sleep(1_000)
      IO.puts("SENDING ASYNC TASK MESSAGE #{n}")
      send(live_view_pid, {:task_message, "Async work chunk #{n}"})
    end)
    # return a small, controlled value
    :ok
  end)
end
```

The first thing we do is call [`self()`](https://hexdocs.pm/elixir/Kernel.html#self/0) and assign the `pid` of the LiveView process to a variable that can be referenced in our async function. This passes the `pid` of the LiveView via a [closure](https://en.wikipedia.org/wiki/Closure_\(computer_programming\)) to our async function.

Before we start the work, we execute the [`AsyncResult.loading()`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.AsyncResult.html#loading/0) function which tells LiveView that our `:async_result` (this could be named anything you want) is now in a loading state.

The “work” being done in our function is looping 5 times, sleeping for 1 second, then sending a message to the LiveView process about the chunk of work we completed.

When the function finishes, we’ll be notified and we’ll exit the `loading` state. That comes later.

## LiveView’s new handle\_async/3 callback

LiveView v0.20 introduced a new [`handle_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_async/3) callback function that fires when the result of a [`start_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#start_async/3) operation is available.

We’ll use this to update our UI when the async process either:

- completes successfully
- completes with an error
- the task process crashes

Pattern matching helps us tell the difference between these three result states.

Something worth pointing out is that the first argument to the callback is the name we gave our task when we started it. This means that we can start **multiple concurrent async tasks** and deal with them all uniquely if we want. That’s pretty cool!

Let’s see what a “success” callback looks like next.

### Successful Completion

When our async function completes successfully, the [`handle_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#c:handle_async/3) callback is fired in our LiveView and we use pattern matching to determine that it succeeded.

If you recall, the result of our async function was `:ok` because we aren’t waiting for the final result in our situation. That `:ok` is what’s coming through in our `handle_async` function.

```elixir
def handle_async(:running_task, {:ok, :ok = _success_result}, socket) do
  socket =
    socket
    |> put_flash(:info, "Completed!")
    |> assign(:async_result, AsyncResult.ok(%AsyncResult{}, :ok))

  {:noreply, socket}
end
```

The `handle_async` function receives `{:ok, function_result}`. So in our case, a success is `{:ok, :ok}`. Yes, that looks odd, but it is valid. 🙂

In the callback, we do two things.

1. Display a flash message that it completed successfully.
2. Clear the “loading” state by updating the `AsyncResult` assigned to `:async_result`.

We use [`AsyncResult.ok/2`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.AsyncResult.html#ok/2) to record that the async process succeeded. We’re passing in two values:

- `%AsyncResult{}` - clears our loading state and resets the `:async_result` value for being called again.
- `:ok` - This would be the result of our function if we were using it that way. Since we don’t have a relevant value to store, we just store `:ok`.

Let’s see how to detect a failure.

### Detecting when the function fails

When the anonymous function passed to `start_async/3` returns an error like `{:error, data}`, it’s actually still a successful result from our function. “Successful” meaning the function didn’t explode but actually returned *something*.

In our situation, we could `send` a message about a failure to the LiveView, or we could let the function return some error data.

If we chose to return the error state and data from our function, it is still received through the `handle_async/3` callback. Our pattern match might look something like this:

```elixir
def handle_async(:running_task, {:ok, {:error, reason}}, socket) do
  socket =
    socket
    |> put_flash(:error, reason)
    |> assign(:async_result, AsyncResult.failed(%AsyncResult{}, reason))

  {:noreply, socket}
end
```

Note we’re still matching on the outer `{:ok, result}` tuple. This is because the function didn’t blow up. But with this error information, we can display a flash message with the reason.

Next, we use the [`AsyncResult.failed/2`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.AsyncResult.html#failed/2) function to track that it failed. Again, we reset the `AsyncResult` to clear the `loading` state.

### Detecting when the function explodes

We’ve touched on the idea that the function isn’t exploding when we detect the success and failure states. Sometimes functions DO explode!

Why might a function explode?

- an exception is raised
- the async process dies (an exception, DB failure, OOM, network connection closed, etc)

In either case, it’s a total failure, but we still want to handle it because one of our goals is that a failure in the task doesn’t crash our LiveView UI.

A task that died still comes through the `handle_async/3` callback. Here’s what the pattern match looks like:

```elixir
def handle_async(:running_task, {:exit, reason}, socket) do
  socket =
    socket
    |> put_flash(:error, "Task failed: #{inspect(reason)}")
    |> assign(:async_result, %AsyncResult{})

  {:noreply, socket}
end
```

The pattern is `{:exit, reason}`. This tells us the processes executing our function died, or exited.

Here we display a flash message about the error with whatever information we have.

Finally, we clear our `:async_result` back to a fresh and shiny blank `%AsyncResult{}`. This clears the loading state so we can try again if desired.

All that’s left to cover is, “how do we cancel a running task?” Let’s explore that next.

## Cancelling the task

One of our goals was to be able to cancel the running async process. How do we do that?

To review, when we called `start_async/3`, we assigned it to `:running_task` in our LiveView’s assigns. We can use this to cancel the process.

Also, we’re using `:async_result` to track the `%AsyncResult{}` struct, specifically to know when we’re `loading` or not.

In our UI, let’s conditionally display a “Start” and “Cancel” button using this “loading” state. The following markup displays a “Start” button when no async process is running and the “Cancel” button when `@async_result.loading`.

```xml
<div class="...">
  <.button :if={!@async_result.loading} phx-click="start">Start</.button>
  <.button :if={@async_result.loading} phx-click="cancel">Cancel</.button>
</div>
```

Now our “Cancel” button only shows up when an async process is running. That was easy!

Next we’ll handle when the “Cancel” button is clicked.

```elixir
def handle_event("cancel", _params, socket) do
  socket =
    socket
    |> cancel_async(:running_task)
    |> assign(:async_result, %AsyncResult{})
    |> put_flash(:info, "Cancelled")

  {:noreply, socket}
end
```

LiveView introduced [`cancel_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#cancel_async/3) just for this purpose!

We call `cancel_async` using the name of the key stored in our assigns from our `start_async` call.

We also clear the `:async_result` to reset the “loading” state. Now our UI correctly reflects that there is no longer a process running and we could start it again if we wanted.

Finally, we add a flash message to display that it was cancelled.

Nice! All cancelled and cleaned up!

We succeeded in:

- starting an async process from our LiveView
- tracking and displaying when the process is running (aka “loading”)
- handling successful completion
- handling async task failures
- handling when an async task crashes
- canceling a running task

We did all that using the Phoenix LiveView’s new **async operations**.

Be sure to check out the [full gist](https://gist.github.com/brainlid/e56a1e04aa8babd8bc6b851d5a07eda5) for putting all the pieces together!

Our situation, where we aren’t waiting for the final result, is outside the normal flow but Phoenix LiveView’s async assigns still works for us. Let’s note what was different.

### We aren’t using assign\_async

We aren’t using the [`assign_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#assign_async/3) function which is what the whole feature is named for! That function works best when we’re loading something in our LiveView’s mount.

Instead, we use [`start_async/3`](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#start_async/3) which gives us more granular control for starting and stopping our async task.

### We still use %AsyncResult{} even when we don’t care about the result

Yup. We still used `%AsyncResult{}` even though we aren’t using it to store our result. It is still a handy way to track our loading state and if our operation was successful or if it failed.

### What we gain from async assigns

Turns out the new [async operations](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html#module-async-operations) are versatile and powerful enough to let us solve a variety of async problems. The feature wraps up the boilerplate needed for:

- starting async tasks
- linking processes
- trapping exits
- cancelling tasks
- tracking when we’re loading
- and more!

All in all, this is a very welcome addition to Phoenix LiveView! Most people will probably use it the prescribed way by fetching async data in a LiveView on mount. But it should give us greater confidence that it’s also flexible enough when our project’s needs take us off the async operations happy path.

Have you given the new features a spin yet?