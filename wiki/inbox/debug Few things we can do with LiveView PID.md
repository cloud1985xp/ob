---
title: "[debug] Few things we can do with LiveView PID"
source: "https://arathunku.com/b/2021/debug-few-things-we-can-do-with-liveview-pid/"
author:
  - "[[@arathunku]]"
published: 2021-05-09
created: 2026-05-05
description: "Let's see what we can do with a PID when debugging Phoenix.LiveView views"
tags:
  - "clippings"
---
This post assumes you know some things about:

- [Phoenix LiveView](https://hex.pm/packages/phoenix_live_view)
- [PID](https://erlang.org/doc/reference_manual/processes.html)
- [:dbg](https://erlang.org/doc/man/dbg.html)

---

Text on a button wasn’t what I expected it to be.

I’ve checked code and there was something like this inside my component: (simplified)

```elixir
~L"""
<% copy = if(@initial, do: "Create and continue", else: "Update") %>
<%= button(copy) %>
"""
```

I’ve got a variable `@initial`. It was a multistep form, and it should have been `true` but somehow, along the way, it was set to `false` or maybe it was never set to `true`, or maybe something else was wrong?

At that point, I started wondering how I can debug this more easily than adding `IEx.pry`, reloading the page and starting over again. It was only happening when I was creating the resource and it was multistep form, not the most pleasant way of debugging.

## How to get Phoenix LiveView PID?

We can get PID in few ways. I’ll list only the methods that I’m using.

- HTML
- [Phoenix LiveDashboard](https://github.com/phoenixframework/phoenix_live_dashboard)
- iex console

## HTML

(in case there’s a problem with loading the tweet)

> I’ve started including @elixirphoenix LiveView PID in HTML comment. It makes debugging 10x easier when you need to send a message to LiveView process or just simply inspect wtf it has in it. #MyElixirStatus

And on screenshots, there’s a snippet where I add:

```html
<!-- pid: <%= raw inspect(self()) %> -->
```

to my `live.html.eex`.

![Screenshot of Reviewsaurus with PID highlighted](https://arathunku.com/b/2021/debug-few-things-we-can-do-with-liveview-pid/20210506-201220_hu_fb8d1de5ca10be9d.png)

On left side is my app, on right side HTML code with PID. This LiveView process controls the view.

## LiveDashboard

We can also find PID easily without adding any special code.

In [Phoenix LiveDashboard](https://github.com/phoenixframework/phoenix_live_dashboard), we can go to “Processes” view and we’re looking for `Phoenix.LiveView.Channel`

![Screenshot of LiveDashboard](https://arathunku.com/b/2021/debug-few-things-we-can-do-with-liveview-pid/20210506-195511_hu_234f5ebe5f628c7a.png)

I’ve got two tabs open so there’re 2 processes.

## Console

This one is simple and basically does what LiveDashboard does under the hood, just without pretty UI:

```elixir
Process.list()
|> Enum.map(& {
  &1,
  Process.info(&1, [:dictionary])
  |> hd()
  |> elem(1)
  |> Keyword.get(:"$initial_call", {})
})
|> Enum.filter(fn {_, process} ->
  process != nil && process != {} &&
==
end)
```

The last line is very verbose so I usually put such methods into my `dev` helper module.

```elixir
defmodule H do
  def live_list do
    Process.list()
    |> Enum.map(& {
      &1,
      Process.info(&1, [:dictionary])
      |> hd()
      |> elem(1)
      |> Keyword.get(:"$initial_call", {})
    })
    |> Enum.filter(fn {_, process} ->
      process != nil && process != {} &&
==
    end)
    |> Enum.map(& elem(&1, 0))
  end
end
```
```elixir
iex(app@127.0.0.1)38> H.live_list()
[#PID<0.1015.0>]
```

Now, we have obtained process PID, and assigned it to beautifully named and descriptive variable `p`.

```elixir
p = H.live_list() |> hd()
```

It will be used throughout the rest of the post, so if you see `p` anywhere - that’s the PID we’re looking into!

I’m mostly interested in two things - inspecting, and interacting with the process from the console.

## Inspecting state

For this, we’re going to use [`:sys.get_state/1`](https://erlang.org/doc/man/sys.html#get_state-1)

```elixir
iex(app@127.0.0.1)31> :sys.get_state(p)
%{
  components: {%{
=>
      "reminder:1ec30c53-b0a2-496c-a828-ce0a6804ebc4",
      %{
        chset: #Ecto.Changeset<action: nil, changes: %{}, errors: [],
         data: #Reviewsaurus.Reminders.Reminder<>, valid?: true>,
         current_user: #Reviewsaurus.Accounts.User<
          id: "4e3a9f23-49e7-45d5-9819-88629e23d228",
          username: "arathunku",
          ...
        >,
        flash: %{},
        initial: false,
        myself: %Phoenix.LiveComponent.CID{cid: 1},
        reminder: #Reviewsaurus.Reminders.Reminder<
          id: "1ec30c53-b0a2-496c-a828-ce0a6804ebc4",
          name: "alles",
          ...
        >
      }, %{changed: %{}}, {78102743095449827113310915107249600236, %{}}}
   },
   %{
     ReviewsaurusWeb.Dashboard.RemindersLive.FormDetailsComponent => %{
       "reminder:1ec30c53-b0a2-496c-a828-ce0a6804ebc4" => 1
     }
   }, 2},
  join_ref: "4",
  serializer: Phoenix.Socket.V2.JSONSerializer,
  socket: #Phoenix.LiveView.Socket<
    assigns: %{
      current_user: #Reviewsaurus.Accounts.User<
        id: "4e3a9f23-49e7-45d5-9819-88629e23d228",
        username: "arathunku",
        ...
      >,
      flash: %{},
      initial: true,
      live_action: :details,
      reminder: #Reviewsaurus.Reminders.Reminder<
        id: "1ec30c53-b0a2-496c-a828-ce0a6804ebc4",
        name: "alles",
        ...
      >,
      timezone: "Etc/UTC"
    },
    changed: %{},
    endpoint: ReviewsaurusWeb.Endpoint,
    id: "phx-FnyLqoxd6Tb3yAIF",
    parent_PID: nil,
    root_PID: #PID<0.1383.0>,
    router: ReviewsaurusWeb.Router,
    view: ReviewsaurusWeb.Dashboard.RemindersLive.Form,
    ...
  >,
  topic: "lv:phx-FnyLqoxd6Tb3yAIF",
  transport_PID: #PID<0.1377.0>,
  upload_names: %{},
  upload_PIDs: %{}
}
```

We can see everything! Internal LiveView.Channel data, our assigns, endpoint, related PIDs, nested components and more.

In my case of a faulty copy on a button, I’m interested in assigns.

Not sure if you notice, but I’ve already spotted why I had a different copy on a button - somehow my `initial` value isn’t updated in nested live component with its own state.

I’ve added this method too to a helper:

```elixir
defmodule H do
  ...

  def state(pid) when is_pid(pid), do: :sys.get_state(pid)
end
```

It’s basically an alias at this point but I find it easier to have all my helpers in one place.

Let’s see what else we can do with `p`.

## Interacting with LiveView Process

Given that we have process’s PID, Erlang VM allows us to for example… replace process’s state!

Example, setting `:foobar` key in assigns to `true`:

```elixir
iex(app@127.0.0.1)> :sys.replace_state(p, & put_in(&1, [:socket, Access.key(:assigns), :foobar], true))
# ...large output ommited
iex(app@127.0.0.1)> H.state(p) |> get_in([:socket, Access.key(:assigns), :foobar])
true
```

`Access.elem(:assigns)` is required because `Phoenix.LiveView.Socket` doesn’t implement Access behavior:

```elixir
iex(app@127.0.0.1)> :sys.replace_state(p, & put_in(&1, [:socket, :assigns, :foobar], true));0
** (ErlangError) Erlang error: {:callback_failed, {:gen_server, :system_replace_state},
  {:error, %UndefinedFunctionError{
    arity: 3, function: :get_and_update, message: nil, module: Phoenix.LiveView.Socket,
    reason: "Phoenix.LiveView.Socket does not implement the Access behaviour"}
  }}
    (stdlib 3.15) sys.erl:160: :sys.replace_state/2
```

Unfortunately, nothing will change on the page after doing that. That’s because no events are actually sent on the websocket connection to the client. Phoenix.LiveView.Channel doesn’t know about our `replace_state` calls, but we can do something else.

## Updating state

Given that `replace_state` wasn’t enough, it won’t forward updates to the page, and I want to avoid digging into [Phoenix.LiveView.Channel](https://github.com/phoenixframework/phoenix_live_view/blob/v0.15.5/lib/phoenix_live_view/channel.ex) internals(I didn’t avoid it, you can) on how to “force” push an update.

My “hack” around that is to add additional helper on dev env. It’s done by default when `use FooWeb, :live_view` is used. All components receive this helper method:

```elixir
@impl true
def handle_info({:_dev_update, f}, socket) do
  {:noreply, f.(socket)}
end
```

and it can be called with: (remember, `p` is our stored earlier in a variable `PID` of the process)

```elixir
iex(app@127.0.0.1)> send(p, {:_dev_update, & Phoenix.LiveView.assign(&1, :initial, true) } )
{:_dev_update, #Function<44.37215449/1 in :erl_eval.expr/5>}
iex(app@127.0.0.1)> send(p, {:_dev_update, & Phoenix.LiveView.assign(&1, :initial, false) } )
{:_dev_update, #Function<44.37215449/1 in :erl_eval.expr/5>}
iex(app@127.0.0.1)>
```

This allows us to update anything in socket’s assigns without a problem. It also sends updates to the page.

You can do more interesting things with it like, replacing current user with another one and seeing how the page would look like for them!

<video controls="" width="100%"><source src="https://arathunku.com/b/2021/debug-few-things-we-can-do-with-liveview-pid/demo.mp4" type="video/mp4"></video>

---

Last thing that I was interested in after I had already fixed my bug was tracing. Can I easily see all events, without `IO.inspect` in ~~thoughtfully selected~~ random places?

## Tracing updates

What events our component receives, what it sends back to the client?

To trace processes, we can use [`:dgb`](https://erlang.org/doc/man/dbg.html), more precisely:

```elixir
:dbg.tracer()
:dbg.p(p)
```

Watch out! This will be extremely verbose! We would also have to remember about closing the trace. My console output was all messed up because it would trace and print everything, including all HTML that’s pushed via websocket connection.

Let’s add another helper function!

```elixir
defmodule H do
  ...

  def trace_component(PID, stop_after_count \\ 1) when is_PID(PID) and is_integer(stop_after_count) do
    :dbg.tracer(
      :process,
      {
        fn
          _msg, ^stop_after_count ->
            IO.inspect("trace done")
            :dbg.stop_clear()

          {:trace, _PID, :send, {:socket_push, _, _}, _}, i ->
            IO.inspect("truncated socket push")
            i + 1
          msg, i ->
            IO.inspect(msg)
            i + 1
        end,
        0
      }
    )

    :dbg.p(PID)
  end
end
```

In `trace_component/2` call above we do few things:

1. Take pid and number of events we want to collect. I enforce this number because otherwise we’d have to remember about manually stopping it, and there’s a lot of going on inside LiveView process!
2. Start tracer and attach it to our process that controls LiveView.Channel
3. We explicitly handle events related to `:socket_push` because they produce a lot of noise, we could further down trim down logged out data if we needed to
4. Print anything else!

This also work remotely within a cluster, but I’ll leave this as an exercise for the reader.

## Watch out

When you’re sending events to Phoenix.LiveView.Channel, you’re actually sending them to the parent component. If there’re nested stateful live components, you need to do dig deeper with updates and most likely define additional filtering when tracing events. Stateful components are still within the same Erlang process.

## Kill process

```elixir
Process.exit(p, :kill)
```

Check if your app handles brief disconnect nicely. This can happen during deployment.

---

There’s more than one way to do what was described in the post, and it’s not extensive list. You could also use [`:observer`](https://erlang.org/doc/man/observer.html) or something else that provides similar functionalities in the console. You could also attach to remote `iex` console and debug on production.

---

If you know about other methods that are a bit friendlier from dev UX perspective - please share them. I’d be more than happy to link/include them in the post.

---

In case you have any comments or questions - feel free to contact me at [email](mailto:m@arathunku.com).