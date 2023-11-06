---
layout: post
title: CircuitPython Type Annotations
date: 2023-11-05 23:09 -0800
tags:
  - Python
  - CircuitPython
  - LanguageServerProtocol
  - KMK
  - Pyright
  - Tools
  - Development
---

Fairly recently, I picked up a new keyboard. Not just _any_ keyboard though. I _finally_ got around to picking up a Corne keyboard. The journey to the specific one I chose is a bit of a story, but that's not what this article is about.

I ended up selecting the [Boardsource Corne](https://www.boardsource.xyz/products/Corne) powered by the [Blok](https://www.boardsource.xyz/products/blok) Rpi-2040 microcontroller. For me, this hit a bunch of sweet spots in terms of price, power, customizability, and ease of use. It's been my daily driver now for several months, and I can't imagine going back.

The default user experience for the Blok Microcontroller on supported keyboards is the [Peg](https://peg.software/) keymap manager (it does more than just keymaps - see the website for more details). In my opinion, Peg is sufficient for a lot of the tasks one would wish to do in terms of layout management. Based on [KMK](http://kmkfw.io/), with some customizations, Peg allows for pretty effective management of keymaps, code blocks, OLED management, LED mappings, etc, etc.

Since I tend to be ... a _little bit weird_, I wanted the generated code to be simpler. I wanted it to be easier for me to read, and understand. I wanted the formatting to look different than what Peg generated. I _could_ manage all this and still use Peg, but at that point, I would be getting no benefit from the Peg software. So, I set out to make my own configurations, including a `boot.py` that suits my needs.

The frustrations of developing those pieces by hand were exacerbated by the default python tooling not having type definitions for CircuitPython types. For any reasonably seasoned CircuitPython developer, the answer may have been simple and obvious - but that shared intuition doesn't appear to be documented anywhere that I could easily find.

## Problem Statement:

Out of the box, python tooling (even for a KMK development environment) does not include type definitions for CircuitPython. This makes inspections, linting, formatting, and warnings inaccurate, even when using something like [Pyright](https://microsoft.github.io/pyright/#/) configured with `useLibraryCodeForTypes`.

## Solution:

The solution is surprisingly simple - Python has something called a [typeshed](https://github.com/python/typeshed), which (as I understand it), is effectively a method for providing type information for target environments to Python tooling.

Assuming the use of [Pipenv](https://github.com/pypa/pipenv) within a project, one can add the CircuitPython types with only two libraries:

```
pipenv install --dev types-circuitpython
pipenv install --dev adafruit-circuitpython-typing
```

With those two packages installed, the static type checker, formatter, linter, LSP server, or whatever tooling you use _should_ pick up the CircuitPython types correctly and begin offering proper typing tooltips and code completion ... and get rid of those pesky false positive warnings!

These type definitions don't need to be present at runtime, which is why they're installed with the `--dev` category. If you're _not_ using `Pipenv`, then you can simply install the packages with `pip`, or whatever system you use to manage python dependencies - just make sure to install them in your Virtual Environment - these types will be useless for regular cPython code.
