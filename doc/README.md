

# hooks - Generic Hooks system for Erlang and Elixir pplications #

Copyright (c) 2015-2016 Benoît Chesneau.

__Version:__ 2.0.0

# hooks

`hooks` is a generic Hooks system for **Erlang** applications. It allows you to
augment your application by adding hooks to your application aka
[Hooking](https://en.wikipedia.org/wiki/Hooking). Hooks can also be [used easily 
with Elixir applications](#usage-in-elixir-applications).

[![Build Status](https://travis-ci.org/barrel-db/hooks.png?branch=master)](https://travis-ci.org/barrel-db/hooks)
[![Hex pm](http://img.shields.io/hexpm/v/hooks.svg?style=flat)](https://hex.pm/packages/hooks)

Main Features are:

- Handle module hooks
- Basic plugin system
- Registered hooks are exported as functions in a dynamically compiled erlang module . It allows us to share the list of registered hooks between every process of your application without message passing. It is also memory efficient and minimize locking.

## Usage in Erlang Applications

Full application API is available in [`hooks`](hooks.md) .

### adding hooks manually

Your application can add hooks using the following methods

- `hooks:reg/{3, 4, 5}` to register a hook from a module
- `hook:unreg/{3, 5, 5}` to unregister a hook.

```
ok = hooks:reg(a, ?MODULE, hook_add, 1, 10),
ok = hooks:reg(a, ?MODULE, hook_add2, 1, 0),
```

### add multiple hooks

- `hook:mreg/1`: to register multiple hooks
- `hooks:munreg/1`: to register multiple hooks

Ex:

```
Hooks = [{a, [{?MODULE, hook1, 0},
              {?MODULE, hook2, 0}]},
         {b, [{?MODULE, hook1, 2},
              {?MODULE, hook2, 2}]},
         {c, [{?MODULE, hook_add, 1},
              {?MODULE, hook_add1, 1}]}],
%% register multiple hooks
ok = hooks:mreg(Hooks),
%% unregister multiple hooks
ok = hooks:munreg(Hooks)
```

### Enable/Disable Plugins

Plugins are simple Erlang applications that exposes hooks. A plugin can be enabled to your application using `hooks:enable_plugin/{1,2}` and disabled using `hooks:disable_plugin/1` . When enabled the application and its dependencies are started (if not already stared) and exposed hooks are registered.

To expose the hooks,  just add them to application environment settings. Example:

```
{application, 'myapp',
 [{description, ""},
  {vsn, "1.0.0"},

  ...

  {env,[
    {hooks, [{a, [{?MODULE, hook1, 0},
                  {?MODULE, hook2, 0}]},
             {b, [{?MODULE, hook1, 2},
                  {?MODULE, hook2, 2}]}]},
    ...
  ]},

  ...

 ]}.
```

> You can specify a patch where to load the application and its dependencies.

### run hooks

You can use the following command to execute hooks

- `hooks:run/2` :run all hooks registered for the HookName.
- `hooks:run_fold/3`: fold over all hooks registered for HookName, and return Acc.
- `hooks:all/2`: execute all hooks for this HookName and return all results
- `hooks:all_till_ok/2`: execute all hooks for the HookName until one return ok or {ok, Val}.
- `hooks:only/2`: call the top priority hook for the HookName and return the result.

### advanced features

#### Internal  hooks

2 internal hooks are exposed

- `init_hooks`: the hook is executed when hooks is started, a function of arity 0 is expected.
- `build_hooks`: the hooks is executed when the list of hooks has changed. A function of arity 1, receiving the list of hooks is expected.

When added to the hooks application environnement, the hooks are immediately available and won't wait for any registered process.

#### wait_for_proc application setting

The `wait_for_proc` application environment settings in the `hooks` application allows you to wait for a specific registered process (example your main application process) to be started before making the hooks available. It means that until the process isn`t registered the beam containing the list of hooks won't be compiled with the list of added hooks.

#### custom start/stop functions

When enabling a plugin the application is generally started like any OTP application. In some cases however you may want to use your own start/stop functions.

To do it create an `Application` module and add to it the functions `start/0` and `stop/0`. `Application:start/0` should return ok or an error if it can't be started.

## Usage in Elixir applications

Add hooks to your mix app by adding hooks to your list of dependencies,

```
[{:hooks, "~> 2.0.0"}]

## [{:hooks, git: "https://github.com/barrel-db/hooks"}]
```

Sample code usage is as follows:

```
defmodule DemoMReg do
    def run do
        Application.ensure_all_started(:hooks)
        hooks = [
                    {:a, [{__MODULE__, :hook1, 1}, {__MODULE__, :hook1, 1}]},
                    {:b, [{__MODULE__, :hook1, 1}, {__MODULE__, :hook1, 1}]}
                ]

        IO.inspect Hooks.mreg(hooks)
        IO.inspect Hooks.find(:b)
        IO.inspect Hooks.all(:b, [1])
    end

    def hook1(args) do
        [ok: args]
    end
end

defmodule DemoReg do
    def run do
        Application.ensure_all_started(:hooks)
        IO.inspect Hooks.reg(:c, __MODULE__, :hook1, 1)
        IO.inspect Hooks.find(:c)
        IO.inspect Hooks.all(:c, [1])
    end

    def hook1(args) do
        [ok: args]
    end
end
```
