---
tags:
  - elixir
  - language
  - phoenix
created: 2025-01-01
updated: 2025-01-23
status: active
---


# Sigils

https://hexdocs.pm/elixir/sigils.html
https://elixirschool.com/en/lessons/basics/sigils

ex:
```
ids = ~w(....) |> Enum.map(&String.to_integer/1)
```

# ETS

https://www.itnota.com/start-observer-erlang-elixir-shell/
:observer.start()

There’s also :erlang.memory()  which will print usage by different items


# Resources

Elixir-Lang
https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html#our-first-project

Elixir School
https://elixirschool.com/en/lessons/basics/functions/

Learn With Me: Elixir
https://inquisitivedeveloper.com/tag/lwm-elixir/

Hexdocs
https://hexdocs.pm/
https://hexdocs.pm/elixir/1.12/Kernel.html#get_in/2
https://hexdocs.pm/mix/main/Mix.Tasks.Format.html
https://hexdocs.pm/elixir/1.13/typespecs.html

## Topics

- Naming: https://hexdocs.pm/elixir/1.12.3/naming-conventions.html
- Library Guidlines: https://hexdocs.pm/elixir/1.12.3/library-guidelines.html
- Stream: https://hexdocs.pm/elixir/Stream.html#with_index/2
- Operators: https://hexdocs.pm/elixir/operators.html
- Mix: 
	- https://hexdocs.pm/mix/Mix.html
	- https://hexdocs.pm/elixir/introduction-to-mix.html
- Kernel: https://hexdocs.pm/elixir/Kernel.html
  - https://hexdocs.pm/elixir/1.12/Kernel.html#apply/2
- Struct: https://hexdocs.pm/elixir/structs.html
- Module: 
	- https://hexdocs.pm/elixir/Module.html
	- https://elixirschool.com/en/lessons/basics/modules#use-8
- Sigils: https://elixir-lang.org/getting-started/sigils.html
- Function
- Guard: https://hexdocs.pm/elixir/patterns-and-guards.html
- Plug: https://hexdocs.pm/plug/Plug.Conn.html
- Testing: 
	- https://hexdocs.pm/phoenix/testing.html
	- https://elixirschool.com/zh-hant/lessons/testing/basics
- Documentation: 
	- https://elixirschool.com/en/lessons/basics/documentation
	- https://hexdocs.pm/elixir/writing-documentation.html
	- https://github.com/elixir-lang/ex_doc
	- https://elixirforum.com/t/using-mermaid-with-ex-doc/40727
- Debugging: 
	- https://hexdocs.pm/elixir/1.16/debugging.html#observer
	- https://elixir-lang.org/getting-started/debugging.html
- Code Point: https://en.wikipedia.org/wiki/Code_point
- Comprehensions: https://hexdocs.pm/elixir/comprehensions.html
- Capture Operator &
- Metaprogramming
	- https://elixirschool.com/en/lessons/advanced/metaprogramming
	- 
### Phoenix
- Contexts: https://hexdocs.pm/phoenix/1.7.0-rc.2/contexts.html
- Router
	- http://spaceisdisorienting.com/singular-resources-in-phoenix
- LiveView
	- https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html
	- https://github.com/phoenixframework/phoenix_live_view
	- https://medium.com/@leandrocesquini/phoenix-liveview-collection-8259f35ff2b0
	- https://medium.com/@traceyonim22/rendering-template-in-phoenix-liveview-3b70e1a51c6e

- Component: 
	- https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html
	- https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html#inputs_for/1-dynamically-adding-and-removing-inputs
	- 
- 
- Form:
	- https://hexdocs.pm/phoenix_html/Phoenix.HTML.Form.html
	- https://github.com/phoenixframework/phoenix_html/blob/v4.1.1/lib/phoenix_html/form.ex

### Ecto
- https://hexdocs.pm/ecto/dynamic-queries.html
- https://ulisses.dev/elixir/2019/05/13/how-to-create-a-ecto-setup-pipeline-with-ecto-3-1-2.html
- Validation
	- https://elixirschool.com/blog/til-ecto-validations-and-constraints
- Manipulate Association
	- https://blog.appsignal.com/2020/11/10/understanding-associations-in-elixir-ecto.html
- Enum Type
	- https://dev.to/mnussbaumer/creating-an-ecto-enum-type-43dh

### Release
- https://hexdocs.pm/mix/1.14/Mix.Tasks.Release.html#module-daemon-mode-unix-like
- https://hexdocs.pm/phoenix/releases.html#ecto-migrations-and-custom-commands
- https://staknine.com/phoenix-database-migrations-elixir-releases/

## Implementation

- Ecto, Basic CRUD
	- https://brooklinmyers.medium.com/ecto-with-phoenix-in-4-minutes-9b7c447055c6
- Pagination
	- https://github.com/mojotech/scrivener
	- https://github.com/mojotech/scrivener_ecto
	- https://brooklinmyers.medium.com/pagination-and-infinite-scroll-in-phoenix-d2a5f9bac5d6
	- https://fullstackphoenix.com/tutorials/pagination-with-phoenix-liveview
	- https://github.com/woylie/flop
- Crawler, Http Client, XML Parser
	- https://github.com/sneako/finch
	- https://hexdocs.pm/req/readme.html
	- https://hexdocs.pm/floki/Floki.html
	- https://www.erlang.org/doc/apps/xmerl/xmerl.html
		- https://gist.github.com/sasa1977/5967224
- Nested Form
	- https://fullstackphoenix.com/tutorials/nested-model-forms-with-phoenix-liveview
	- https://www.bcat.eu/blog/elixir-nested-changesets-with-phoenix/
	- https://dockyard.com/blog/2024/03/12/dynamically-add-and-remove-embedded-item-inputs-without-javascript
	- https://kobrakai.de/kolumne/one-to-many-liveview-form
	- https://github.com/LostKobrakai/one-to-many-form/blob/main/lib/one_to_many/groceries_list.ex

## Blogs

- https://dashbit.co/blog

# To Read
- https://medium.com/very-big-things/towards-maintainable-elixir-the-development-process-205ee257c109
- https://selleo.com/blog/learn-elixir-from-zero-to-a-testing-hero

## Books
- https://pragprog.com/titles/phoenix14/programming-phoenix-1-4/
- https://www.packtpub.com/product/mastering-elixir/9781788472678
  - https://github.com/packtpublishing/mastering-elixir

## Mix
https://elixir-lang.org/getting-started/mix-otp/introduction-to-mix.html
https://hexdocs.pm/mix/1.12/Mix.Task.html 
https://hexdocs.pm/mix/Mix.Tasks.Release.html
https://hexdocs.pm/phoenix/Mix.Tasks.Phx.Gen.Schema.html

### Enum
https://hexdocs.pm/elixir/1.13/Enum.html#filter/2
Enum vs Streams
https://elixir-lang.org/getting-started/enumerables-and-streams.html

Application
https://hexdocs.pm/elixir/Application.html

System
https://hexdocs.pm/elixir/1.12/System.html

Environment Variables
https://mbuffa.github.io/tips/20210916-elixir-environment-variables/

OptionParse
https://hexdocs.pm/elixir/1.12.3/OptionParser.html#parse/2

### recompile
https://hexdocs.pm/iex/IEx.Helpers.html#recompile/0

## Cond, Case and If-Else
https://elixir-lang.org/getting-started/case-cond-and-if.html

## With Statement
https://www.openmymind.net/Elixirs-With-Statement/

## String, Long String
- https://stackoverflow.com/questions/46095870/whats-the-best-way-to-build-long-strings-in-elixir

## Map / Keyword List / Struct

- https://elixir-lang.org/getting-started/keywords-and-maps.html#maps
- https://medium.com/@brucepomeroy/accepting-optional-options-in-elixir-65e7eaed11ac
- https://adolfont.medium.com/exercises-on-elixir-maps-keyword-lists-sets-and-structs-4cba2d00e9b3

### Map vs Struct

Map, dynamic access and strict access
> m = %{name: "Mike", age: 15}
> m.name
> m.foo -> Error

Struct uses strict access by default, unless you @derive [Access]

## Pattern Matching

- https://inquisitivedeveloper.com/lwm-elixir-23/


## File System

- https://code.tutsplus.com/tutorials/working-with-file-system-in-elixir--cms-28869

## Import, Require, Alias, Use

- https://curiosum.com/blog/alias-import-require-use-in-elixir
- https://stackoverflow.com/questions/28491306/elixir-use-vs-import

## Behavior / Dynamic Dispatch

- https://dnlserrano.dev/2019/12/21/behaviours-and-dynamic-dispatch.html

## ExUnit (Testing) / Doctest / Typespec
https://hexdocs.pm/ex_unit/1.12/ExUnit.html
https://hexdocs.pm/mix/1.12/Mix.Tasks.Test.html
https://hexdocs.pm/ex_unit/ExUnit.DocTest.html#content
https://elixirschool.com/en/lessons/testing/basics#test-mocks-4

https://elixirforum.com/t/referencing-files-from-tests-dir-is-evaluated-at-compile-time-so-paths-can-break/28230
https://tech.nextroll.com/blog/dev/2018/03/28/elixir-stubs-for-tests.html

https://semaphoreci.com/community/tutorials/a-practical-guide-to-test-doubles-in-elixir

### Use ByPass to Mock HTTP Request
https://elixirschool.com/en/lessons/testing/bypass

### Typespec
- https://hexdocs.pm/elixir/typespecs.html#the-string-type
- https://elixirschool.com/zh-hant/lessons/advanced/typespec/

### Run Mix Task in Release Application?
Actually it's not mix task, just provide module to run the process you want using "eval"
https://hexdocs.pm/mix/Mix.Tasks.Release.html#module-one-off-commands-eval-and-rpc
https://stackoverflow.com/questions/37472766/running-mix-tasks-in-production

## Publish
https://hex.pm/docs/publish

## build executable CLI: use "escript"
> mix escript.build
https://medium.com/blackode/writing-the-command-line-application-in-elixir-78a8d1b1850

## CLI
https://github.com/rizafahmi/elixirdose-cli

## Cross Platform CLI
https://github.com/burrito-elixir/burrito

## ESpec
https://github.com/antonmi/espec
https://dev.mikamai.com/2016/02/23/testing-with-espec-and-elixir/

# Library

https://github.com/peburrows/goth

## Http

- hackney
  - https://github.com/benoitc/hackney
- HTTPoision
  - https://github.com/edgurgel/httpoison
- Mint
  - https://elixir-lang.org/blog/2019/02/25/mint-a-new-http-library-for-elixir/
  - https://scoutapm.com/blog/how-to-use-mint-an-awesome-http-library-for-elixir-part-01
- Finch
  - https://blog.appsignal.com/2020/07/28/the-state-of-elixir-http-clients.html

## ETS and persistent_term

https://hexdocs.pm/elixir/erlang-term-storage.html
https://stackoverflow.com/questions/21973760/how-to-identify-the-exact-memory-size-of-an-ets-table

persistent_term has higher performance for reading, but low writing
- https://www.erlang.org/doc/man/persistent_term.html
- https://stackoverflow.com/questions/65735371/what-are-the-differences-between-ets-persistent-term-and-process-dictionaries

# Project / Tool / Library

- Static Code Analysis:
 - https://github.com/rrrene/credo
 - https://hexdocs.pm/dialyzex/Mix.Tasks.Dialyzer.html
   - https://github.com/jeremyjh/dialyxir
- Mimic: Mocking library
- https://github.com/multiprocessio/dsq
  - Commandline tool for quering JSON/Excel/CSV files
- Coverage Report
  - https://github.com/parroty/excoveralls
- Prometheus / OpenTelemetry
  - https://github.com/akoutmos/prom_ex

## Livebook
https://livebook.dev/
https://github.com/livebook-dev/livebook/blob/main/README.md

## IOT
https://www.nerves-project.org/


# Erlang
- https://www.erlang.org/doc/apps/stdlib/index.html

dbg
- https://github.com/fishcakez/dbg
- https://elixirforum.com/t/how-to-use-the-dbg-tracer-in-elixir/44702/5

## NIF
Native Implemented Function
- https://www.erlang.org/doc/system/nif.html
- https://medium.com/@jlouis666/erlang-dirty-scheduler-overhead-6e1219dcc7

### Rust and Rustler
Rust / Cargo Installation
https://doc.rust-lang.org/cargo/getting-started/installation.html

Rust in Elixir: rustler
https://github.com/rusterlium/rustler
https://github.com/philss/rustler_precompiled
https://fly.io/phoenix-files/elixir-and-rust-is-a-good-mix/
https://mainmatter.com/blog/2023/02/01/using-rust-crates-in-elixir/
https://discord.com/blog/using-rust-to-scale-elixir-for-11-million-concurrent-users

# Script Histories

pid = spawn(fn -> 1 + 2 end)

Process.alive?(pid)
Process.info(pid)
Process.info(pid, :reductions)
Process.list()
Process.list() |> Enum.sort_by(& Process.info(&1, :reductions), :desc) |>  hd |> Process.info()

pid = spawn(fn -> receive do
    "ping" -> IO.puts("pong")
  end
end)

send(pid, "ping")

Link & Monitor

process 1 -> link -> process 2
crash 1 -> crash 2

pid = spawn_link(fn -> raise "crash!" end)
if pid process crashed, the parent(ex: iex) also crash

  spawn
  send
  receive
  spawn_link

# Task

Task.start(fn -> raise "crash!" end)
Task.start_link(fn -> raise "crash!" end)

wrapper of spawn/spawn_link?



# Agent

- https://elixir-lang.org/getting-started/mix-otp/agent.html

Task.start_link(fn -> loop(%{}) end)

def loop(map) do
  receive do

    {:get, key, caller} ->
      send caller, Map.get(map, key)
      loop(map)
    {:put key, value} ->
      loop(Map.put(map, key, value))
  end
end

Agent => wrapper of above codes

## ex:
Agent.get
Agent.update

# OTP / GenServer 


https://hexdocs.pm/elixir/1.12/GenServer.html#module-example
https://elixirschool.com/zh-hant/lessons/advanced/otp_concurrency

Generic Server

call vs cast
cast -> no care response


# Supervisor
https://hexdocs.pm/elixir/1.12/Supervisor.html#module-module-based-supervisors

start child process
to manage child process
ex:
- restart some child process base on certain conditions
- shot down all child processed when ...


{:ok, pid } = Supervisor.start_link(__MOUDULE__, :ok, opts)

strategy options
:one_for_one
:one_for_all
:rest_for_one


