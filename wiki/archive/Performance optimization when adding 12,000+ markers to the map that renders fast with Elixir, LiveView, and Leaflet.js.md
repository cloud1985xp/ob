---
title: "Performance optimization when adding 12,000+ markers to the map that renders fast with Elixir, LiveView, and Leaflet.js"
source: "https://dev.to/azyzz/performance-optimization-when-adding-12000-markers-to-the-map-that-renders-fast-with-elixir-liveview-and-leafletjs-54pf"
author:
  - "[[aziz abdullaev]]"
published: 2023-12-04
created: 2026-04-07
description: "The task was to have a map that renders the location of 12,000+ sites by adding a circle marker and... Tagged with elixir, webdev, leaflet, javascript."
tags:
  - "clippings"
---
The task was to have a map that renders the location of 12,000+ sites by adding a circle marker and popup window when clicked. And it must be very fast, ideally so fast that it seems it’s instantaneous. In this article, I will share my implementation with Elixir, LiveView, and Leaflet.js along with optimization techniques I came up with.

If we were to zoom out, There are two main tasks that we need to do:

1. Fetch data from database and pass it to the frontend
2. Render the data on a map

First part is handled by the server while rendering the map happens on the client side. Because there is a lot of data we need to fetch and render, optimizations are needed on both server and client sides.

To implement the map using Elixir, LiveView, and Leaflet.js, we need to do the following:

1. Elixir fetches data from the database and passes it to the LiveView
2. LiveView receives the data and passes it to the client (JavaScript (JS) Hook with `push_event`)
3. JS handles the received data by creating a map with Leaflet.js and adding marker

My configs are:

- Elixir 1.15.7
- OTP 26
- LiveView 0.20.1
- Phoenix 1.7.10
- Phoenix LiveView 0.18.18
- Ecto 3.11
- Dockerized PostgreSQL
- All running on my MacBook Air M2 8GB Ram

Let’s do the initial setup.

When the user opens the page, LiveView process is created, web socket connection established, and we can query the data we want in the LiveView. Our LiveView will contain a map created with Leaflet.js. Thanks to the great JS interoperability (interop) that comes with LiveView, we easily can do that.  
 A little side note if English is not your first language (which is my case). JS interoperability means JS integration, basically how we can use JS with LiveView. Okay, let’s continue.

Here is our initial LiveView.  

```
defmodule CoolMapWeb.HomeLive do

  def mount(_, _, socket) do
    {:ok, socket}
  end

 def render(assigns) do
    ~H"""
    <div class="container mx-auto w-[80%]">
      <div class="w-full h-[500px] p-20" id="map" phx-hook="Map"></div>
    </div>
    """
  end
```

Nothing interesting going on here except for having a `<div>` with `phx-hook=“Map”` — that is how JS interop is implemented in LiveView (see the official docs). Now, let’s create a hook that will allow us to use Leaflet.js to create the map.

To install Leaflet.js through NPM, go to /assets/vendor direction and run `npm install leaflet`

Inside of our app.js file:  

```
import L from "../vendor/node_modules/leaflet";

const Map = {
    mounted() {
        const map = L.map("map").setView([41, 69], 2);
        L.tileLayer(
            "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}",
            {
                attribution:
                    "Tiles &copy; Esri &mdash; Esri, DeLorme, NAVTEQ",
                maxZoom: 16,
            }
        ).addTo(map);

    },
};

const Hooks = {
    Map,
};

// add Hooks here
let liveSocket = new LiveSocket("/live", Socket, { params: { _csrf_token: csrfToken }, hooks: Hooks })
```

Here is the result:

[![ ](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn555rck5txiwmy2xqtxh.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fn555rck5txiwmy2xqtxh.png)

Let’s do the simplest implementation of the task. When the LiveView loads, it will request a data from a Context, then push it to the Leaflet.js. To pass the data from LiveView to Leaflet.js Hook, we need to use `push_event` function on the LiveView and `handle_event` on the Hook.

Here is the function from Context that will query the projects with coordinates for us:  

```
def list_projects do
    query = from p in Project, where: not is_nil(p.latitude), order_by: p.id and not is_nil(p.longitude)
    Repo.all(query)
  end
```

We will request the data on LiveView’s mount function and pass it to the JS side:  

```
def mount(_, _, socket) do
    projects = Context.list_projects() # array [%Project{}, %Project{}]
    socket = 
    socket 
    |> push_event("add_project”, %{projects: projects)

    {:ok, socket}
  end
```

Note: the push\_event in this case will raise an error because the payload must not contain the structs. We will fix this issue later.

To receive the data on JS side, we can either add window listener or add event handler to the hook. I went with option #2.  

```
const Map = {
    mounted() {
        const map = L.map("map").setView([41, 69], 2);
        L.tileLayer(
            "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}",
            {
                attribution:
                    "Tiles &copy; Esri &mdash; Esri, DeLorme, NAVTEQ",
                maxZoom: 16,
            }
        ).addTo(map);

        // event handler
        this.handleEvent("add_project", (project) => {
        // add markers
        });

    },
};
```

Now, let’s discuss the interesting part: optimizations.

Elixir is built on top of Erlang and is fantastic when it comes to concurrent programming and many processes. When I ran query on my laptop (both dockerized postgreSQL and Phoenix application), I get the following results:

[![ ](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4izcss6e7xhb4djb23pp.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F4izcss6e7xhb4djb23pp.png)

Running the simple query function `list_projects()` takes around 187 ms when measured with:timer.tc function. My DB has around 54,000 entries but only around 12,300 have both latitude and longitude. So, `list_projects()` results in around 12,300 items in an array of results.

When I used `list_projects()` on my laptop, passed it to the LiveView, and added markers for the map, my laptop just froze and took around 10+ seconds to recover. Although it takes only 187ms when I run the function on `iex` terminal, there are several reasons why it takes so long to actually produce a page and map with markers. One of the main reasons is the fact that JS code runs on the client side, although LiveView is a process living on the server. So the solution would be to pass the data to the JS side in smaller chunks so that browser does not crash and/or freeze.

On top of that, using concurrent programming in Elixir, we can asynchronously run multiple functions that will fetch the data. If we were to have chunks of 6,000 entries, then we would spawn two processes: one queries data for entries 1st to 5,999th, and the second from 6,000th to 12,000th. We can make chunk even smaller, I decided to have chunks with the size of 2,000 entires. When we run multiple asynchronous functions, we do not know which function will finish running first, so we need a mechanism of passing data to the LiveView then to the JS whenever asynchronous function returns the data from database.

How can we do that? Use the most important feature of LiveView: LiveView is a process, so it has its own PID. So, we will pass a PID of the LiveView to querying async functions, and functions will `Kernel.send()` the data to the LiveView.

Here is my implementation of the Context function that will receive the PID of the LiveView, asynchronously do DB queries, and send it to the LiveView.  

```
def stream_projects(pid, offset \\ 0, batch_size \\ 2000, total_records \\ 14000) do
    batches = div(total_records, batch_size)

    async_stream =
      Task.async_stream(0..(batches - 1), fn batch ->
        current_offset = offset + batch * batch_size

        query =
          from p in Project,
            where: not is_nil(i.latitude) and not is_nil(i.longitude),
            limit: ^batch_size,
            offset: ^current_offset

        case Repo.all(query) do
          [] ->
            :done

          result ->
            send(pid, {:data_received, result})
        end
      end)

    async_stream
    |> Stream.run()

    :done
  end
```

In `stream_projects` function, we create a stream of async functions that fetch the data 0th to 2000th, then 2000th to 4000th, etc, then send the data to the PID of the LiveView. (Please refer to official documentation on Streams and async\_streams)

Below is the screenshot of performance measurement of `stream_projects()`. Note: in my project, function name is different, but the content of the function is identical to `stream_projects()`. It only takes 66ms to run the function.

[![ ](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5rx67amifb3n5g7z1jie.png)](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2F5rx67amifb3n5g7z1jie.png)

In LiveView, we need to call the `stream_projects` on mount, then add `handle_info` function to receive the data and pass it to the JS Hook.  

```
def mount(_, _, socket) do
    Projects.stream_all_projects(self())

    {:ok, socket}
  end

 # ADD THIS FUNCTION
  def handle_info({:data_received, data}, socket) do
    socket =
      Enum.reduce(data, socket, fn %Project{} = project, socket ->
        socket
        |> push_event("add_project", form_project_map(project))
      end)

    {:noreply, socket}
  end

  defp form_project_map(%Project{} = project) do
    %{
      latitude: project.latitude,
      longitude: project.longitude,
      name: project.name,
    }
  end
```

In `handle_info`, we are creating an accumulated socket with projects passed to the JS hook individually. We are also creating a map from the %Project{} structure.

Now, let’s add a full event handler on JS hook that will be able to create a marker on project’s location. By default, markers are rendered as SVGs and added to the map and HTML, when we have lots of new markers we add a lot of new nodes to the DOM (which is very costly performance wise). The key optimization with Leaflet.js is to use `Canvas` renderer for markers so that nodes are not created and added to the DOM (FYI [https://stackoverflow.com/questions/37043791/plotting-140k-points-in-leafletjs](https://stackoverflow.com/questions/37043791/plotting-140k-points-in-leafletjs)).  

```
const Map = {
    mounted() {
        const map = L.map("map").setView([41, 69], 2);
        L.tileLayer(
            "https://server.arcgisonline.com/ArcGIS/rest/services/Canvas/World_Light_Gray_Base/MapServer/tile/{z}/{y}/{x}",
            {
                attribution:
                    "Tiles &copy; Esri &mdash; Esri, DeLorme, NAVTEQ",
                maxZoom: 16,
            }
        ).addTo(map);

        let myRenderer = L.canvas({ padding: 0.5 });
        this.handleEvent(“add_project", (project) => {

            let marker = L.circleMarker(
                [project.latitude, project.longitude],
                {
                    renderer: myRenderer,
                    radius: 1,
                    width: 0,
                    color: “#ef4444”,
                    fillColor: “#ef4444”,
                    fillOpacity: 0.8,
                    fill: true,
                }
            )
                .addTo(map);
                .bindPopup(\`<p>${project.name}</p>\`);

    },
};
```

Here is the result of the map that renders around 12,000 points from the cold start:

![ ](https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fg2tatvpzjyo4gsu46tqx.gif)

Discussion and notes:

- Error handling on `stream_projects()` does not exist
- The total number of projects is known and does not change frequently — this allows us to manually set the `total_records` to 14,000 in `stream_projects`.
- `total_records` number is larger than exact number of entries, so we may be doing some extra queries.
- Possible improvement on JS hook is to add async handling and addition of markers to the map
- The actual loading speed of the map when deployed will depend on bandwidth, DB query latency, and client’s hardware.

This is my first article on programming, take it with the grain of salt. Please reach out if you have any corrections or suggestions for my article.

Part 2 is available here [https://dev.to/azyzz/network-optimization-for-sending-lot-of-data-from-liveview-to-client-using-pushevent-2nl](https://dev.to/azyzz/network-optimization-for-sending-lot-of-data-from-liveview-to-client-using-pushevent-2nl)[MongoDB](https://dev.to/mongodb)Promoted

[![Build gen AI apps that run anywhere with MongoDB Atlas](https://media2.dev.to/dynamic/image/width=775%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fi.imgur.com%2FCjuXF8e.png)](https://www.mongodb.com/cloud/atlas/lp/try3?utm_campaign=display_devto-webdev_pl_flighted_atlas_tryatlaslp_prosp_gic-null_ww-all_dev_dv-all_eng_leadgen&utm_source=devto&utm_medium=display&utm_content=aipowered-v1&bb=239014)

## Build gen AI apps that run anywhere with MongoDB Atlas

MongoDB Atlas bundles vector search and a flexible document model so developers can build, scale, and run gen AI apps without juggling multiple databases. From LLM to semantic search, Atlas streamlines AI architecture. Start free today.