FileSystem
=========

A file change watcher wrapper based on [fs](https://github.com/synrc/fs)

## System Support

- Mac fsevent
- Linux and FreeBSD inotify
- Windows inotify-win

NOTE: On Linux and FreeBSD you need to install inotify-tools.

## Usage

Put `file_system` in the `deps` and `application` part of your mix.exs

``` elixir
defmodule Excellent.Mixfile do
  use Mix.Project

  def project do
  ...
  end

  defp deps do
    [
      { :file_system, "~> 0.1.0", only: :test },
    ]
  end
  ...
end
```


### Subscription API

You can spawn a worker and subscribe to events from it:

```elixir
{:ok, pid} = FileSystem.start_link(dirs: ["/path/to/some/files"])
FileSystem.subscribe(pid)
```

or

```elixir
{:ok, pid} = FileSystem.start_link(dirs: ["/path/to/some/files"], name: :my_monitor_name)
FileSystem.subscribe(:my_monitor_name)
```

The pid you subscribed from will now receive messages like

```
{:file_event, worker_pid, {file_path, events}}
```
and
```
{:file_event, worker_pid, :stop}
```

### Example with GenServer

```elixir
defmodule Watcher do
  use GenServer

  def start_link(args) do
    GenServer.start_link(__MODULE__, args)
  end

  def init(args) do
    {:ok, watcher_pid} = FileSystem.start_link(args)
    FileSystem.subscribe(watcher_pid)
    {:ok, %{watcher_pid: watcher_pid}}
  end

  def handle_info({:file_event, ^watcher_pid, {path, events}}, %{watcher_pid: watcher_pid}=state) do
    # YOUR OWN LOGIC FOR PATH AND EVENTS
    {:noreply, state}
  end

  def handle_info({:file_event, ^watcher_pid, :stop}, %{watcher_pid: watcher_pid}=state) do
    # YOUR OWN LOGIC WHEN MONITOR STOP
    {:noreply, state}
  end
end
```


## Tweaking behaviour via listener extra arguments

For each platform, you can pass extra arguments to the underlying listener process via the `listener_extra_args` option.

Here is an example to get instant notifications on file changes for Mac OS X:

```elixir
FileSystem.start_link(dirs: ["/path/to/some/files"], listener_extra_args: "--latency=0.0")
```

See the [fs source](https://github.com/synrc/fs/tree/master/c_src) for more details.

## List Events from Backend

```shell
iex > FileSystem.known_events
```
