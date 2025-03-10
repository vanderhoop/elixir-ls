# Elixir Language Server (ElixirLS)

The Elixir Language Server provides a server that runs in the background, providing IDEs, editors, and other tools with information about Elixir Mix projects. It adheres to the [Language Server Protocol](https://github.com/Microsoft/language-server-protocol), a standard for frontend-independent IDE support. Debugger integration is accomplished through the similar [VS Code Debug Protocol](https://code.visualstudio.com/docs/extensionAPI/api-debugging).

## This is now the main elixir-ls repo

The [elixir-lsp](https://github.com/elixir-lsp)/[elixir-ls](https://github.com/elixir-lsp/elixir-ls) repo began as a fork when the original repo at [JakeBecker](https://github.com/JakeBecker)/[elixir-ls](https://github.com/JakeBecker/elixir-ls) became inactive for an extended period of time. So we decided to start an active fork to merge dormant PR's and fix issues where possible. We also believe in an open and shared governance model to share the work instead of relying on one person to shoulder the whole burden.

The original repository has now been deprecated in favor of this one. Any IDE extensions that use ElixirLS should switch to using this repository.

## Features

- Debugger support
- Automatic, incremental Dialyzer analysis
- Automatic inline suggestion of @specs based on Dialyzer's inferred success typings
- Inline reporting of build warnings and errors
- Documentation lookup on hover
- Go-to-definition
- Code completion
- Code formatter
- Find references to functions and modules (Thanks to @mattbaker)
- Quick symbol lookup in file (Thanks to @mattbaker)
- Quick symbol lookup in workspace and stdlib (both Elixir and erlang) (@lukaszsamson)

![Screenshot](images/screenshot.png?raw=true)

Note: On first run Dialyzer will build a PLT cache which will take a considerable amount of CPU time (usually 10+ minutes). After that is complete the CPU usage will go back to normal. Alternatively instead of waiting you can disable Dialyzer in the settings.

## IDE plugins

| IDE          | Plugin                                                                        | Support                                                               |
| ------------ | ----------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| Emacs        | [eglot](https://github.com/joaotavora/eglot)                                  |                                                                       |
| Emacs        | [lsp-mode](https://github.com/emacs-lsp/lsp-mode)                             | Supports debugger via [dap-mode](https://github.com/yyoncho/dap-mode) |
| Kakoune      | [kak-lsp](https://github.com/kak-lsp/kak-lsp)                                 | [Limitations](https://github.com/kak-lsp/kak-lsp/#limitations)        |
| Kate         | [built-in LSP Client plugin](https://kate-editor.org/post/2020/2020-01-01-kate-lsp-client-status/) | Does not support debugger                        |
| Neovim       | [coc.nvim](https://github.com/neoclide/coc.nvim)                              | Does not support debugger                                             |
| Neovim       | [nvim-dap](https://github.com/mfussenegger/nvim-dap)                          | Supports debugger only                                                |
| Neovim       | [nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)                    | Does not support debugger                                             |
| Nova         | [nova-elixir-ls](https://github.com/raulchedrese/nova-elixir-ls)              |                                                                       |
| Sublime Text | [LSP-elixir](https://github.com/sublimelsp/LSP-elixir)                        | Does not support debugger                                             |
| Vim/Neovim   | [ALE](https://github.com/w0rp/ale)                                            | Does not support debugger or @spec suggestions                        |
| Vim/Neovim   | [elixir-lsp/coc-elixir](https://github.com/elixir-lsp/coc-elixir)             | Does not support debugger                                             |
| Vim/Neovim   | [vim-lsp](https://github.com/prabirshrestha/vim-lsp)                          | Does not support debugger                                             |
| VS Code      | [elixir-lsp/vscode-elixir-ls](https://github.com/elixir-lsp/vscode-elixir-ls) | Supports all ElixirLS features                                        |

Feel free to create and publish your own client packages and add them to this list!

## Detailed Installation Instructions

How you install ElixirLS depends on your editor.

For VSCode install the extension: https://marketplace.visualstudio.com/items?itemName=JakeBecker.elixir-ls

<details>
  <summary>Emacs Installation Instructions</summary>

  Download the latest release:
  https://github.com/elixir-lsp/elixir-ls/releases/latest and unzip it into a
  directory (this is the directory referred to as the
  `"path-to-elixir-ls/release"` below)

  If using `lsp-mode` add this configuration:
  ```elisp
    (use-package lsp-mode
      :commands lsp
      :ensure t
      :diminish lsp-mode
      :hook
      (elixir-mode . lsp)
      :init
      (add-to-list 'exec-path "path-to-elixir-ls/release"))
  ```

  For eglot use:
  ```elisp
  (require 'eglot)

  ;; This is optional. It automatically runs `M-x eglot` for you whenever you are in `elixir-mode`
  (add-hook 'elixir-mode-hook 'eglot-ensure)

  ;; Make sure to edit the path appropriately, use the .bat script instead for Windows
  (add-to-list 'eglot-server-programs '(elixir-mode "path-to-elixir-ls/release/language_server.sh"))
  ```
</details>

## Supported versions

Elixir:

- 1.11.0 minimum

Erlang:

- OTP 22 minimum

Installing Elixir and Erlang from [ASDF](https://github.com/asdf-vm/asdf) is generally recommended so that you can have different projects using different versions of Elixir without having to change your system-installed version. ElixirLS will detect and use the version of Elixir and Erlang that you have configured in asdf.

## Debugger support

ElixirLS includes debugger support adhering to the [Debug Adapter Protocol](https://microsoft.github.io/debug-adapter-protocol/) which is closely related to the Language Server Protocol. At the moment, line breakpoints and function breakpoints are supported.

When debugging in Elixir or Erlang, only modules that have been "interpreted" (using `:int.ni/1` or `:int.i/1`) will accept breakpoints or show up in stack traces. The debugger in ElixirLS automatically interprets all modules in the Mix project and dependencies prior to launching the Mix task, so you can set breakpoints anywhere in your project or dependency modules.

Currently there is a limit of 100 breakpoints.

### Debugging tests and `.exs` files

In order to debug modules in `.exs` files (such as tests), they must be specified under `requireFiles` in your launch configuration so they can be loaded and interpreted prior to running the task. For example, the default launch configuration for "mix test" in the VS Code plugin looks like this:

```
{
  "type": "mix_task",
  "name": "mix test",
  "request": "launch",
  "task": "test",
  "taskArgs": [
    "--trace"
  ],
  "startApps": true,
  "projectDir": "${workspaceRoot}",
  "requireFiles": [
    "test/**/test_helper.exs",
    "test/**/*_test.exs"
  ]
}
```

In order to debug a single test or a single test file it is currently necessary to modify `taskArgs` and make sure no other tests are required in `requireFiles`.

```
{
  "type": "mix_task",
  "name": "mix test",
  "request": "launch",
  "task": "test",
  "taskArgs": ["tests/some_test.exs:123"],
  "startApps": true,
  "projectDir": "${workspaceRoot}",
  "requireFiles": [
    "test/**/test_helper.exs",
    "test/some_test.exs"
  ]
}
```

### Phoenix apps

Use the following launch config to debug phoenix apps

```
{
  "type": "mix_task",
  "name": "phx.server",
  "request": "launch",
  "task": "phx.server",
  "projectDir": "${workspaceRoot}"
}
```

Please make sure that `startApps` is not set to `true` as it prevents phoenix from starting correctly. On the other hand phoenix tests expects that the apps are already started so in that case set it to `true`.

### NIF modules limitation

Please note that due to `:int` limitation NIF modules cannot be interpreted and need to be excluded via `excludeModules` option. This option can be also used to disable interpreting for some modules when it is not desirable e.g. when performance is not satisfactory.

```
{
  "type": "mix_task",
  "name": "mix test",
  "request": "launch",
  "task": "test",
  "taskArgs": ["--trace"],
  "projectDir": "${workspaceRoot}",
  "requireFiles": [
    "test/**/test_helper.exs",
    "test/**/*_test.exs"
  ],
  "excludeModules": [
    ":some_nif",
    "Some.SlowModule"
  ]
}
```

### Function breakpoints

Function breakpoints will break on the first line of every clause of the specified function. The function needs to be specified as MFA (module, function, arity) in the standard elixir format, e.g. `:some_module.function/1` or `Some.Module.some_function/2`. Breaking on private functions is not supported via function breakpoints.

### Conditional breakpoints

Break conditions are supported and evaluate elixir expressions within the context set of breakpoints. See also limitations on Expression evaluator for further details.

### Hit conditions

An expression that evaluates to integer can be used to control how many hits of a breakpoint are ignored before the process is stopped.

### Log points

When log message is set on a breakpoint the debugger will not break but instead log a message to standard output (as required by Debug Adapter Protocol specification). The message may contain interpolated expressions in `{}`, e.g. `my_var is {inspect(my_var)}` and will be evaluated in the context of the process. Special characters `{` and `}` can be emitted with escape sequence `\{` and `\}`. As of Debug Adapter Protocol specification version 1.51, log messages are not supported on function breakpoints.

### Expression evaluator

An expression evaluator is included in the debugger. It evaluates elixir expressions in the context of a process stopped on a breakpoint. All bound variables are accessible (no support for attributes as those are compile time). Please note that there are limitations due to `:int` operating on beam instruction level. The binding returns multiple versions of variables in Static Singe Assignment with no indication which one is valid in the current elixir scope. A heuristic is used that selects the highest versions but it does not behave correctly in all cases, e.g. in

```elixir
a = 4
if true do
  a = 5
end
some
```

when a breakpoint is reached on line with `some`, the last bound value for `a` seen by expression breakpoint evaluator is 5 (should be 4).

## Automatic builds and error reporting

Builds are performed automatically when files are saved. If you want this to happen automatically when you type, you can turn on "autosave" in your IDE.

Starting in Elixir 1.6, Mix compilers adhere to the [Mix.Task.Compiler](https://hexdocs.pm/mix/master/Mix.Task.Compiler.html) behaviour and return their error and warning diagnostics in a standardized way. Errors and warnings will be shown inline in your code as well as in the "Problems" pane in the IDE. If you're using an earlier version of Elixir, you'll need to look at the text log from the extension to see the errors and warnings.

## Dialyzer integration

ElixirLS will automatically analyze your project with [Dialyzer](http://erlang.org/doc/apps/dialyzer/dialyzer_chapter.html) after each successful build. It maintains a "manifest" file in `.elixir_ls/dialyzer_manifest` that stores the results of the analysis. The initial analysis for a project can take a few minutes, but after that's completed, modules are re-analyzed only if necessary, so subsequent analyses are typically very fast -- often less than a second. It also looks at your modules' abstract code to determine whether they reference any modules that haven't been analyzed and includes them automatically.

You can control which warnings are shown using the `elixirLS.dialyzerWarnOpts` setting in your project or IDE's `settings.json`. Find available options in Erlang [docs](http://erlang.org/doc/man/dialyzer.html) at section "Warning options".

To disable Dialyzer completely, set `elixirLS.dialyzerEnabled` to false.

Sometimes dialyzer can get stuck with incorrect/no longer applying warnings. It's best to restart the language server in that case.

ElixirLS's Dialyzer integration uses internal, undocumented Dialyzer APIs, and so it won't be robust against changes to these APIs in future Erlang versions.

## Code completion

ElixirLS bundles an advanced code completion provider. The provider builds on [Elixir Sense](https://github.com/elixir-lsp/elixir_sense) library and utilizes two main mechanisms. The first one is reflection - getting information about compiled modules from Erlang and Elixir APIs. The second one is AST analysis of the current text buffer. While reflection gives precise results, it is not well suited for on demand completion of symbols from the currently edited file. The compiled version is likely to be outdated or the file may not compile at all. AST analysis helps in that case but it has its limitations. Unfortunately it is infeasible to be 100% accurate, especially with Elixir being a metaprogramming heavy language.

The completions include:

- keywords
- special form snippets
- functions
- macros
- modules
- variables
- struct fields (only if the struct type is explicitly stated or can be inferred from the variable binding)
- atom map keys (if map keys can be inferred from variable binding)
- attributes
- types (in typespecs)
- behaviour callbacks (inside the body of implementing module)
- protocol functions (inside the body of implementing module)
- keys in keyword functions arguments (if defined in spec)
- function returns (if defined in spec)

## Workspace Symbols

With Dialyzer integration enabled ElixirLS will build an index of symbols (modules, functions, types and callbacks). The symbols are taken from the current workspace, all dependencies and stdlib (Elixir and erlang). This feature enables quick navigation to symbol definitions.

## Troubleshooting

Basic troubleshooting steps:
* Restart your editor (which will restart ElixirLS)
* After stopping your editor, remove the entire `.elixir_ls` directory, then restart your editor
  * NOTE: This will cause you to have to re-run the entire dialyzer build

If your code doesn't compile in ElixirLS, it may be because ElixirLS compiles code with `MIX_ENV=test` (by default). So if you are missing some configuration in the test environment, your code may not compile.

If you get an error like the following immediately on startup:

```
[Warn  - 1:56:04 PM] ** (exit) exited in: GenServer.call(ElixirLS.LanguageServer.JsonRpc, {:packet, %{...snip...}}, 5000)
    ** (EXIT) no process: the process is not alive or there's no process currently associated with the given name, possibly because its application isn't started
```

and you installed Elixir and Erlang from the Erlang Solutions repository, you may not have a full installation of erlang. This can be solved with `sudo apt-get install esl-erlang`. Originally reported in [#208](https://github.com/elixir-lsp/elixir-ls/issues/208).

On fedora if you only install the elixir package you will not have a full erlang installation, this can be fixed by running `sudo dnf install erlang` (reported in [#231](https://github.com/elixir-lsp/elixir-ls/issues/231))

If you are using Emacs with lsp-mode there's a possibility that you have set the
wrong directory as the project root (especially if that directory does not have
a `mix.exs` file). To fix that you should remove the project and re-initialize:
https://github.com/elixir-lsp/elixir-ls/issues/364#issuecomment-829589139

## Known Issues/Limitations

* `.exs` files don't return compilation errors
* "Fetching n dependencies" sometimes get stuck (remove the `.elixir_ls` directory to fix)
* "Go to definition" does not work within the `scope` of a Phoenix router
* On first launch dialyzer will cause high CPU usage for a considerable time
* Dialyzer does not pick up changes involving remote types (https://github.com/elixir-lsp/elixir-ls/issues/502)

## Building and running

Run `mix compile`, then `mix elixir_ls.release -o <release_dir>`. This builds the language server and debugger as a set of `.ez` archives and creates `.sh` and `.bat` scripts to launch them.

If you're packaging these archives in an IDE plugin, make sure to build using the minimum supported OTP version for the best backwards-compatibility. Alternatively, you can use a [precompiled release](https://github.com/elixir-lsp/elixir-ls/releases).

### Local setup

Because ElixirLS may get launched from an IDE that itself got launched from a graphical shell, the environment may not
be complete enough to run or even find the correct Elixir/OTP version. The wrapper scripts try to configure `asdf-vm`
if available, but that may not be what you want or need. Therefore, prior to executing Elixir, the script will source
`$XDG_CONFIG_HOME/elixir_ls/setup.sh` (e.g. `~/.config/elixir_ls/setup.sh`), if available. The environment variable
`ELS_MODE` is set to either `debugger` or `language_server` to help you decide what to do inside the script, if needed.

Note: for windows the local setup script path is `%APPDATA%/elixir_ls/setup.bat` (which is often `C:\Users\<username>\AppData\Roaming\elixir_ls`)

Note: It is important that the setup script not print any output. On linux this might be
accomplished by adding `>/dev/null` and/or `2>/dev/null` at the end of any line that produces
output, and for a windows batch script you will want `@echo off` at the top and `>nul` on every
line.

## Environment variables

* `ELS_INSTALL_PREFIX`: The folder where the language server got installed to. If set eq. through a wrapper script, it makes maintaining multiple versions/instances on the same host much easier. If not set or empty, a heuristic will be used to discover the install location.

## Acknowledgements and related projects

ElixirLS isn't the first frontend-independent server for Elixir language support. The original was [Alchemist Server](https://github.com/tonini/alchemist-server/), which powers the [Alchemist](https://github.com/tonini/alchemist.el) plugin for Emacs. Another project, [Elixir Sense](https://github.com/elixir-lsp/elixir_sense), builds upon Alchemist and powers the [Elixir plugin for Atom](https://github.com/msaraiva/atom-elixir) as well as another VS Code plugin, [VSCode Elixir](https://github.com/fr1zle/vscode-elixir). ElixirLS uses Elixir Sense for several code insight features. Credit for those projects goes to their respective authors.

## License

ElixirLS source code is released under Apache License 2.0.

See [LICENSE](LICENSE) for more information.
