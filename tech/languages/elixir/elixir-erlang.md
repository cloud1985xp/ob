---
tags:
  - elixir
  - language
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Elixir / Erlang

Created: 2021年6月6日 上午3:04

## Tai-an Shares

[https://paper.dropbox.com/published/Elixir-guide-djBxRytxYSyO3bDGqg6kTt5](https://paper.dropbox.com/published/Elixir-guide-djBxRytxYSyO3bDGqg6kTt5)

[https://onedrive.live.com/?authkey=!AOoXe13axsweKaM&cid=58015BCB002E6AC8&id=58015BCB002E6AC8!37279&parId=58015BCB002E6AC8!35794&o=OneUp](https://onedrive.live.com/?authkey=%21AOoXe13axsweKaM&cid=58015BCB002E6AC8&id=58015BCB002E6AC8%2137279&parId=58015BCB002E6AC8%2135794&o=OneUp)

## Topics

The Road to 2 Million Websocket Connections

[The Road to 2 Million Websocket Connections in Phoenix](https://www.phoenixframework.org/blog/the-road-to-2-million-websocket-connections)

## OTP / GenServr

[TIL GenServer's `handle_continue/2`](https://elixirschool.com/blog/til-genserver-handle-continue/)

[Elixir School](https://elixirschool.com/zh-hant/lessons/advanced/otp-supervisors/)

## Mix Application

config/config.exs

config/{dev,prod,test}.exs

預設的 application port 是 8000，在 config 中可以設定

## Libraries

[akira/exq](https://github.com/akira/exq/tree/master/config)

JTW

[Implement token Authentication with Phoenix.Token](https://dev.to/onpointvn/implement-jwt-authentication-with-phoenix-token-n58)

[Joken Overview — Joken v2.6.0](https://hexdocs.pm/joken/introduction.html)

[Configuration — Joken v2.6.0](https://hexdocs.pm/joken/configuration.html)

[Blog · Elixir School](https://elixirschool.com/blog/jwt-auth-with-joken/)

# Basics

true, false, nil are all atom

equals to :true, :false, :nil

you just can skip :

constants are also atom

```elixir
is_atom(IEx.Helpers)
```

List vs Tuple

linear vs indexed(pre-calculated)

length vs size

Keyword lists are the same as lists of two-element tuples:

### Map

when pattern matching, map will always match on a **subset** of the given value ⇒ as long as the keys in the pattern exists in the given map ⇒ empty map matches all maps

### Function

```elixir
def func(...) do
end

def func(...) when … do

end

def func(...), do: # one-liners
```

& operator to **capture** function

```elixir
f = &Math.sum/1
is_function(f)
```

& capture operation as shortcut to create function

```elixir
func =&(&1 + 1)
```

default arguments

```elixir
def func(arg \\ 1) do
end
```

If a function with default arguments has multiple clauses

有預設參數的函式，有多種同形子句時，要提前單獨宣告

```elixir
def join(a, b \\ nil, sep \\'' # 單獨宣告

def join(a, b, _sep) when is_nil(b) do
end

def join(a, b, sep) do
end
```

&+/2 ⇒ capture :+ function/2

Enum & String

### File, IO

背後其實是用 process 實現的

當 IO.write(file, “something”) 第一個參數其實是 pid

也就是 file = [IO.open](http://IO.open) 其實是產生(spawn)一個 process 得到的 pid

IO#write 是對那個 process send message 來執行操作

### Require

require macros

### Import

- import functions and require macros
- discouraged, prefer to use alias

### Use

- For extension
- inject any code
- compiled to
    - require + **using**
- side effected

## Module Attribute

Uses function as module attribute

會先被 compile，如果後面的 function call 使用到 module attribute，是用已經 compile 後的 value

```elixir
defmodule Foo
  @url = URL.parse("https://www.google.com")
  def bar() do
    SomeHttpClient.get(@url)
  end
end

實際上會是

defmodule Foo
  @url = URL.parse("https://www.google.com")
  def bar() do
    SomeHttpClient.get(%{
      authority: "www.google.com",
      host: "....",
      ...
    })
  end
end

# 所以也不能在 module attribute 使用同一個 module define 的 function，
# 因為該 module 還沒 compile
```

Struct

- bare map, can use Map.#func to operate it
- But Not shared implementation of map, like enumerable, or any additional implementation

Protocol

### Comprehensions

Generator

```elixir
.... n <- [1,2,3] ...
```

← 左邊是 pattern matching, non-matching patterns are **ignored**

## LiveState

[LiveState.Channel — live_state v0.6.1](https://hexdocs.pm/live_state/LiveState.Channel.html#summary)

[Building an Embeddable Web App with LiveState, Elixir, and Lit](https://launchscout.com/blog/embedded-web-apps-with-livestate)

phoenix example

[phx-live-state](https://launchscout.github.io/phx-live-state/)

[https://github.com/launchscout/phx-live-state](https://github.com/launchscout/phx-live-state)

[https://github.com/launchscout/live_state](https://github.com/launchscout/live_state)

JS Example

[phx-live-state/LiveState.ts at 0309dad7cf77dc9b6d8ddf3d95c5d233cc9553fa · launchscout/phx-live-state](https://github.com/launchscout/phx-live-state/blob/0309dad/src/LiveState.ts#L92)

[livestate-comments/LivestateComments.ts at main · launchscout/livestate-comments](https://github.com/launchscout/livestate-comments/blob/main/src/LivestateComments.ts)

### Channel

[Channels — Phoenix v1.7.2](https://hexdocs.pm/phoenix/channels.html)

## Live View

[Phoenix.LiveView - Phoenix LiveView v0.17.11](https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.html)

Build Multiplayer Games

[Building Multiplayer Games With LiveView](https://burritalks.io/talks/dorian-karter-building-multiplayer-games-with-liveview/)

WebSocket Case

[The Road to 2 Million Websocket Connections in Phoenix - Phoenix Blog](https://www.phoenixframework.org/blog/the-road-to-2-million-websocket-connections)

## Process

[Process — Elixir v1.12.3](https://hexdocs.pm/elixir/1.12/Process.html)

## Erlang Library

[Erlang libraries](https://elixir-lang.org/getting-started/erlang-libraries.html)

:downcase.func 是 Erlang Library

- Symbol 小寫 呼叫 方法

ex:

```elixir
:io.format("Hello World ~10.3f~n", [:math.pi])
```

:io, :math 都是 Erlang Lib

:ets 也是

## Ets / Persistent_term

[Erlang -- ets](https://www.erlang.org/doc/man/ets.html#info-1)

[Erlang 項式儲存 (ETS) · Elixir School](https://elixirschool.com/zh-hant/lessons/storage/ets)

[Erlang -- persistent_term](https://www.erlang.org/doc/man/persistent_term.html)

### Ecto

Dynamic Repo

[Replicas and dynamic repositories — Ecto v3.10.1](https://hexdocs.pm/ecto/replicas-and-dynamic-repositories.html)

Schemaless Query

[Schemaless queries — Ecto v3.10.1](https://hexdocs.pm/ecto/schemaless-queries.html)

## Up, Running, Release, Deployment

[Up and Running — Phoenix v1.7.2](https://hexdocs.pm/phoenix/up_and_running.html)

[Configuration and releases](https://elixir-lang.org/getting-started/mix-otp/config-and-releases.html)

[Introduction to Deployment — Phoenix v1.7.2](https://hexdocs.pm/phoenix/deployment.html)

## Debugging

IO.inspect binding()

```elixir
require IEx
IEx.pry

break! SomeModule.somemethod/2

當執行 #somemethod 時會進到中斷點
> whereami
> whereami 20
> continue
```

Debugger and Observer

[Erlang libraries](https://elixir-lang.org/getting-started/erlang-libraries.html)

- GUI

## Behavior

In Elixir, the `term`
 type is a shortcut to represent any type

## Kernel

Kernel#apply

~= #send in ruby

# Writing Test

[mix test - Mix v1.12.3](https://hexdocs.pm/mix/1.12/Mix.Tasks.Test.html)

[ExUnit best practices](https://blog.lelonek.me/exunit-best-practices-2b3a8a194f1d)

[Introduction to Writing Tests in Elixir](https://medium.com/mindvalley-technology/introduction-to-writing-tests-in-elixir-e21b17e8d049)

## Test with Double / Mock

- dependency inject
- run in parallel

[Mocking in Elixir: Comparison between Mox, Mockery, Mimic, Syringe, and Lightweight DI](https://dev.to/calvinsadewa/mocking-in-elixir-comparison-between-mox-mockery-mimic-syringe-and-lightweight-di-3d5h)

[A Practical Guide to Test Doubles in Elixir - Semaphore](https://semaphoreci.com/community/tutorials/a-practical-guide-to-test-doubles-in-elixir)

### Mox

José: What are mocks?

The [wikipedia definition](https://en.wikipedia.org/wiki/Mock_object) is excellent: mocks are simulated entities that mimic the behavior of real entities in controlled ways. I will emphasize this later on but I always consider “mock” to be a noun, never a verb.

[Mox](https://elixirschool.com/en/lessons/testing/mox)

[Mox - Mox v1.0.2](https://hexdocs.pm/mox/Mox.html#module-example)

[Mocks and explicit contracts " Plataformatec Blog](http://blog.plataformatec.com.br/2015/10/mocks-and-explicit-contracts/)

[Functional Mocks with Mox in Elixir](https://blog.carbonfive.com/functional-mocks-with-mox-in-elixir/)

### Mock

[https://github.com/jjh42/mock](https://github.com/jjh42/mock)

### Mimic

## Elixir x AI

[Numerical Elixir (Nx)](https://github.com/orgs/elixir-nx/repositories)

[https://github.com/elixir-nx/nx](https://github.com/elixir-nx/nx)

[https://github.com/elixir-nx/axon](https://github.com/elixir-nx/axon)

[https://github.com/elixir-nx/explorer](https://github.com/elixir-nx/explorer)

### BumbleBee

[Announcing Bumblebee: GPT2, Stable Diffusion, and more in Elixir - Livebook.dev The Livebook Blog](https://news.livebook.dev/announcing-bumblebee-gpt2-stable-diffusion-and-more-in-elixir-3Op73O)

[https://github.com/elixir-nx/bumblebee](https://github.com/elixir-nx/bumblebee)

Example:

[bumblebee/text_classification.exs at main · elixir-nx/bumblebee](https://github.com/elixir-nx/bumblebee/blob/main/examples/phoenix/text_classification.exs)

## NIF

[Erlang -- NIFs](https://www.erlang.org/doc/tutorial/nif.html)

## Rust

[Polars, lightning-fast DataFrame library](https://www.pola.rs/)

[https://github.com/rusterlium/rustler](https://github.com/rusterlium/rustler)

[Rustler - Using Rust crates in Elixir - Mainmatter](https://mainmatter.com/blog/2023/02/01/using-rust-crates-in-elixir/)