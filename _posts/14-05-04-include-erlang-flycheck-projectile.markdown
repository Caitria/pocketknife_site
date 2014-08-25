---
layout: post
title: "The -include erlang directive with emacs+flycheck[+projectile]."
description: "Getting emacs' flycheck to play nicely with the -include preprocessor command. Using projectile to make it shine."
tags: [erlang, emacs, flycheck, projectile]
mixpanel: include_flycheck
---

### The Problem

Over the last two months, I've been working with
[Spatch](http://spatch.co/) to implement an email based worker system
in Erlang. More on that soon. Along the way,
[flycheck](https://github.com/flycheck/flycheck), the
pan-syntax-checker library for emacs, has been a wonderful
companion. It works *almost* perfectly with Erlang.

The *almost* is that flycheck doesn't specify any additional `include`
directories. If you're following the common Erlang application
organization of header files in `include/`, and source files in
`src/`, then flycheck running on a source file will fail to find any
included header files. The `-include(...).` line, along with any lines
of code that depend on a macro defined in the header file, will be
marked as an error.

<!--more-->

### The Solution

Yup, just tell flycheck to add the `include/` directory to the include
file search path.

Digging into `flycheck.el`, a checker is defined for each supported
language. Here's Erlang's:

    (flycheck-define-checker erlang
      "An Erlang syntax checker using the Erlang interpreter."
      :command ("erlc" "-o" temporary-directory "-Wall" source)
      :error-patterns
      ((warning line-start (file-name) ":" line ": Warning:" (message) line-end)
       (error line-start (file-name) ":" line ": " (message) line-end))
      :modes erlang-mode)

I'm impressed by how simple and declarative flycheck makes this: when
in one of the `:modes`, run the given `:command`, and collect errors
using the definitions in `:error-patterns`. I <3 flycheck.

For Erlang, the command is just `erlc`:

    erlc -o ${temporary-directory} -Wall ${source}

> note: I mustachified the command because it makes it more
> readable. I might be wrong.

This runs the compiler, sending the output to a `temporary-directory`
which `flycheck` will be kind enough to create, and using the
`source`, which is the file currently being edited. To add an include
file search directory, `erlc` accepts the `-I` flag.  This'll give us
a command of:

    erlc -I ../include -o ${temporary-directory} -Wall ${source}

But, this is brittle: it only works for files in folders that are at
the level of `include`. bbatsov's always excellent
[Projectile](https://github.com/bbatsov/projectile) comes to the
rescue here the function `projectile-project-root`, which does
what it says on the tin.

Without further ado, here is my Erlang syntax checker:

    (flycheck-define-checker erlang
      "An Erlang syntax checker using the Erlang interpreter."
      :command ("erlc" "-I" (eval (format "%s/include" (projectile-project-root)))
	        "-o" temporary-directory "-Wall" source)
      :error-patterns
      ((warning line-start (file-name) ":" line ": Warning:" (message) line-end)
       (error line-start (file-name) ":" line ": " (message) line-end))
      :modes erlang-mode)


For later: getting flycheck to work with
[dialyzer](http://www.erlang.org/doc/man/dialyzer.html), erlang's type
annotation extensions.