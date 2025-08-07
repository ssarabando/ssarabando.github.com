---
layout: post
title: Linux Mint, Mason and OmniSharp
comments: False
---

# {{ page.title }}

_2025, August 8th_

So, in my [last note]({{ site.baseurl }}/2025/07/31/mason-csharpier-lazyvim-linux-mint.html), I mentioned that I had been coding in .NET 8 without major problems except for being able to use a LSP.

I finally got around to poke around the Mason installation and it turns out the problem is just the script that starts up the server calling `omnisharp` when it should be calling `OmniSharp`...

I do not have much time today, so a quick:

```bash
cd ~/.local/share/nvim/mason/bin
ln ~/.local/share/nvim/mason/packages/omnisharp/OmniSharp omnisharp
```

... fixes the problem.
