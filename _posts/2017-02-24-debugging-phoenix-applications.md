---
layout: post
title: "Debugging Phoenix Applications"
tags: ["phoenix", "elixir", "debug", "pry", "byebug"]
---

The goal is to drop a line like `binding.pry` (or `byebug`) into a phoenix
app to inspect state.

Require IEx at the top of the file you want to debug.

```elixir
require IEx
```

Place the debugger.

```elixir
IEx.pry
```

Run your test or server.

`iex -S mix test web/controllers/user_controller_test.exs`

`iex -S mix phoenix.server`

When execution hits the debug line, you'll see a prompt.

```
Request to pry #PID<0.460.0> at test/controllers/user_controller_test.exs:10

        Enum.each(User, &Repo.insert!(&1))

        IEx.pry

        response = build_conn

Allow? [Yn]
```

Running without the `iex -S` prefix shows a warning when execution hits the
debug line.

```
Cannot pry #PID<0.283.0> at test/controllers/user_controller_test.exs:10. Is anIEx shell running?
```

Be sure to prefix your mix command with `iex -S mix ...` to avoid this.

After sitting in the debugger for 15 seconds, I got this error:

```
20:41:07.390 [error] Postgrex.Protocol (#PID<0.261.0>) disconnected: ** (DBConnection.ConnectionError) owner #PID<0.292.0> timed out because it owned the connection for longer than 15000ms
** (EXIT from #PID<0.292.0>) killed
```

[This page documents the solution](https://github.com/elixir-ecto/ecto/blob/master/lib/ecto/adapters/sql/sandbox.ex).

Postgres connections timeout after 15000 miliseconds by default. Override that configuration by setting the timeout to 10 minutes.

`config/config.exs`

```elixir
config :my_app, MyApp.Repo,
  ownership_timeout: 1000 * 60 * 10
```

Sitting at a breakpoint for more than 60 seconds earns a new error.

```
** (ExUnit.TimeoutError) test timed out after 60000ms. You can change the timeout:

       1. per test by setting "@tag timeout: x"
       2. per case by setting "@moduletag timeout: x"
       3. globally via "ExUnit.start(timeout: x)" configuration
       4. or set it to infinity per run by calling "mix test --trace"
          (useful when using IEx.pry)

     Timeouts are given as integers in milliseconds.
```

I chose to use the `--trace` option. Add it to your iex command.

`iex -S mix test --trace web/controllers/user_controller_test.exs`

Once you're done at a breakpoint, use the `respawn` command to continue
execution.

