---
layout: post
title: Visual Studio Code, Pylint and gettext's _() function
comments: False
---

# {{ page.title }}

_10 Mar 2018_

I love Visual Studio Code. It's an awesome little brother for the VS2017 Pro that I use at work.
I especially like using it to write Python but one of its linters (Pylint) has been annoying me to no end because of gettext's `_()` function.

Since `_` is installed in the built-in namespace by `gettext.GNUTranslations.install()` and is not imported in any modules, PyLint is always complaining that the `_` variable is undefined which is really annoying.

As usual, the solution is easy once you find the right documentation: just use the `--additional-builtins` option when calling Pylint (version 0.7.0 and above).

That means that you have to add the following lines to your user or workspace settings:

```json
"python.linting.pylintArgs": [
    "--additional-builtins",
    "_"
]
```

And that's it.

#### References

* [Pylint and gettext. How to reconcile?](https://lists.logilab.org/pipermail/python-projects/2006-April/000714.html)
* [What's New in Pylint 0.7.0?](https://pylint.readthedocs.io/en/latest/whatsnew/changelog.html#what-s-new-in-pylint-0-7-0)
* [Pylint features](https://pylint.readthedocs.io/en/latest/technical_reference/features.html#id36)