---
tags:
  - elixir
  - genserver
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Using GenServer in Elixir Applications

## Introduction

When developing an Elixir application, you may encounter scenarios where you need to manage state and handle concurrent requests. One of the most popular ways to do this in Elixir is by using GenServers. GenServers are a core part of Elixir's OTP (Open Telecom Platform) library and provide a simple way to build concurrent, fault-tolerant systems.

## What is GenServer?

GenServer is a behaviour in Elixir's OTP library that provides a set of callback functions for building robust, concurrent systems. GenServers are stateful processes that can receive and send messages like any other process in Elixir. When you create a GenServer, you define a set of callback functions that will be called when certain events occur, such as when a message is received or when the process is started or stopped.

## How to use GenServer?

To use GenServer in your Elixir application, you first need to define a module that implements the GenServer behaviour. This module should define the callback functions that will be called by the GenServer. The most important callback function is `handle_call/3`, which is called when the GenServer receives a synchronous request. The `handle_call/3` function should return a tuple containing the response to the request and the new state of the GenServer.

Once you have defined your GenServer module, you can start a new instance of the GenServer by calling the `GenServer.start_link/3` function. This function takes three arguments: the name of the module implementing the GenServer behaviour, the initial state of the GenServer, and an optional list of options.

To send a message to a GenServer, you can use the `GenServer.call/2` function. This function sends a synchronous request to the GenServer and waits for a response. If the GenServer fails to respond within a certain timeout period, the `GenServer.call/2` function will raise a timeout error.

## Conclusion

GenServer is a powerful tool for building concurrent, fault-tolerant systems in Elixir. By using GenServer, you can easily manage state and handle concurrent requests in your applications. If you're new to Elixir or concurrency in general, GenServer is a great place to start. With its simple API and robust error-handling capabilities, GenServer is a must-have tool for any serious Elixir developer.

## Example

Let's take a look at a simple example of how to use GenServer in Elixir. In this example, we'll create a simple counter that can be incremented and decremented using GenServer.

```
defmodule Counter do
  use GenServer

  def start_link(initial_count) do
    GenServer.start_link(__MODULE__, initial_count, name: __MODULE__)
  end

  def init(initial_count) do
    {:ok, initial_count}
  end

  def handle_call({:increment, value}, _from, count) do
    {:reply, count + value, count + value}
  end

  def handle_call({:decrement, value}, _from, count) do
    {:reply, count - value, count - value}
  end
end

```

In this example, we define a `Counter` module that implements the GenServer behaviour. We define two callback functions, `handle_call/3` for synchronous requests and `init/1` for initializing the state of the GenServer.

The `handle_call/3` function takes three arguments: the request, the sender of the request, and the state of the GenServer. The function pattern matches on the request to determine whether to increment or decrement the counter, and then returns a tuple containing the response and the new state of the GenServer.

To start a new instance of the `Counter` GenServer, we can call the `Counter.start_link/1` function, passing in an initial count value. We can then use the `Counter.call/2` function to send messages to the GenServer and receive responses.

```
iex> {:ok, pid} = Counter.start_link(0)
{:ok, #PID<0.152.0>}

iex> Counter.call(pid, {:increment, 5})
5

iex> Counter.call(pid, {:decrement, 3})
2

iex> Counter.call(pid, {:increment, 2})
4

```

As you can see, the `Counter` GenServer is able to manage state and handle concurrent requests in a safe and reliable way.

## Conclusion

In this article, we've seen how to use GenServer in Elixir applications to manage state and handle concurrent requests. We've seen that GenServer is a powerful tool for building robust, fault-tolerant systems, and that it's relatively easy to use once you understand its API. If you're new to Elixir or concurrency in general, I highly recommend taking the time to learn GenServer and its related tools in the OTP library. With GenServer, you can build highly concurrent systems that are resilient to failures and able to handle large numbers of requests with ease.

# Using GenServer in Elixir Applications

## Introduction

When developing an Elixir application, you may encounter scenarios where you need to manage state and handle concurrent requests. One of the most popular ways to do this in Elixir is by using GenServers. GenServers are a core part of Elixir's OTP (Open Telecom Platform) library and provide a simple way to build concurrent, fault-tolerant systems.

## What is GenServer?

GenServer is a behaviour in Elixir's OTP library that provides a set of callback functions for building robust, concurrent systems. GenServers are stateful processes that can receive and send messages like any other process in Elixir. When you create a GenServer, you define a set of callback functions that will be called when certain events occur, such as when a message is received or when the process is started or stopped.

## How to use GenServer?

To use GenServer in your Elixir application, you first need to define a module that implements the GenServer behaviour. This module should define the callback functions that will be called by the GenServer. The most important callback function is `handle_call/3`, which is called when the GenServer receives a synchronous request. The `handle_call/3` function should return a tuple containing the response to the request and the new state of the GenServer.

Once you have defined your GenServer module, you can start a new instance of the GenServer by calling the `GenServer.start_link/3` function. This function takes three arguments: the name of the module implementing the GenServer behaviour, the initial state of the GenServer, and an optional list of options.

To send a message to a GenServer, you can use the `GenServer.call/2` function. This function sends a synchronous request to the GenServer and waits for a response. If the GenServer fails to respond within a certain timeout period, the `GenServer.call/2` function will raise a timeout error.

## Example

Let's take a look at a more complex example of how to use GenServer in an Elixir application. In this example, we'll create a simple key-value store that can be used to store and retrieve values using GenServer.

```
defmodule KeyValueStore do
  use GenServer

  def start_link do
    GenServer.start_link(__MODULE__, %{})
  end

  def get(key) do
    GenServer.call(__MODULE__, {:get, key})
  end

  def set(key, value) do
    GenServer.cast(__MODULE__, {:set, key, value})
  end

  def handle_call({:get, key}, _from, state) do
    {:reply, Map.get(state, key), state}
  end

  def handle_cast({:set, key, value}, state) do
    {:noreply, Map.put(state, key, value)}
  end
end

```

In this example, we define a `KeyValueStore` module that implements the GenServer behaviour. We define two callback functions, `handle_call/3` for synchronous requests and `handle_cast/2` for asynchronous requests.

The `handle_call/3` function takes three arguments: the request, the sender of the request, and the state of the GenServer. The function pattern matches on the request to determine whether to get a value or set a value, and then returns a tuple containing the response and the new state of the GenServer.

The `handle_cast/2` function takes two arguments: the request and the state of the GenServer. This function is used for handling asynchronous requests, such as when setting a value in the key-value store.

To start a new instance of the `KeyValueStore` GenServer, we can call the `KeyValueStore.start_link/0` function. We can then use the `KeyValueStore.get/1` and `KeyValueStore.set/2` functions to retrieve and store values in the key-value store.

```
iex> {:ok, pid} = KeyValueStore.start_link
{:ok, #PID<0.178.0>}

iex> KeyValueStore.set(:name, "John")
:ok

iex> KeyValueStore.get(:name)
"John"

iex> KeyValueStore.set(:age, 30)
:ok

iex> KeyValueStore.get(:age)
30

```

As you can see, the `KeyValueStore` GenServer is able to store and retrieve values in a safe and reliable way.

## Conclusion

In this article, we've seen how to use GenServer in Elixir applications to manage state and handle concurrent requests. We've seen that GenServer is a powerful tool for building robust, fault-tolerant systems, and that it's relatively easy to use once you understand its API. We've also seen a simple example of how to use GenServer to build a counter, and a more complex example of how to use GenServer to build a key-value store. If you're new to Elixir or concurrency in general, I highly recommend taking the time to learn GenServer and its related tools in the OTP library. With GenServer, you can build highly concurrent systems that are resilient to failures and able to handle large numbers of requests with ease.