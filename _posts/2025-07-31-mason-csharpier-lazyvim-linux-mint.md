---
layout: post
title: Linux Mint, LazyVim, Mason, Conform, CSharpier and dotnet
comments: false
---

# {{ page.title }}

_2025, July 31st_

Almost two full weeks after moving from Windows 10 to Linux Mint.

I've been coding in .NET 8 without major problems besides being unable to use a LSP (maybe some other post if I ever find out why).

But working without a formatter has been a pain point too, so I decided to figure out why CSharpier (installed with Mason) was not working.

[Turns out](https://github.com/LazyVim/LazyVim/discussions/4596), [LazyVim](https://www.lazyvim.org/) uses [Conform](https://github.com/stevearc/conform.nvim) to do the formatting and it defaults to using LSP formatting unless it is configured for a file type.

Knowing that, I wrote a first configuration override in `~/.config/nvim/lua/plugins/conform.lua`:

```lua
return {
  "stevearc/conform.nvim",
  opts = {
    formatters_by_ft = {
      cs = { "csharpier" },
    },
  },
}
```

That fixed the problem of the formatter not being called but revealed another: it was always throwing an error when saving.

<pre>
Formatter 'csharpier' error: Could not execute because the specified command or file was not found.
Possible reasons for this include:
* You misspelled a built-in dotnet command.
* You intended to execute a .NET program, but dotnet-csharpier does not exist.
* You intended to run a global tool, but a dotnet-prefixed executable with this name could not be found on the PATH.
</pre>

Looking at the default Conform configuration (found in `~/.local/share/nvim/lazy/conform.nvim/lua/conform/formatters`), CSharpier has this:

```lua
---@type conform.FileFormatterConfig
return {
  meta = {
    url = "https://github.com/belav/csharpier",
    description = "The opinionated C# code formatter.",
  },
  command = "dotnet",
  args = { "csharpier", "--write-stdout" },
  stdin = true,
}
```

Comparing the above with the CSharpier documentation it is clear that the [`format`](https://csharpier.com/docs/CLI) command is missing.
So I added a few more lines to my configuration:

```lua
return {
  "stevearc/conform.nvim",
  opts = {
    formatters = {
      csharpier = {
        command = "dotnet",
        args = { "csharpier", "format", "--write-stdout" },
        stdin = true,
      },
    },
    formatters_by_ft = {
      cs = { "csharpier" },
    },
  },
}
```

Even after that it still was failing with the same error so I ended up installing CSharpier globally (by running `dotnet tool install csharpier --global` in a terminal) and changing the configuration again so that the command is now `csharpier` instead of `dotnet`:

```lua
return {
  "stevearc/conform.nvim",
  opts = {
    formatters = {
      csharpier = {
        command = "csharpier",
        args = { "format", "--write-stdout" },
        stdin = true,
      },
    },
    formatters_by_ft = {
      cs = { "csharpier" },
    },
  },
}
```

And with all that I have an auto-formatter working even without a LSP.
