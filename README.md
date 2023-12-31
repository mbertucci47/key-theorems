# key-theorems

An experimental LaTeX package to use [`amsthm`](https://www.ctan.org/pkg/amsthm)
with a key-value interface. Meant to emulate most of the functionality of [`thmtools`](https://www.ctan.org/pkg/thmtools)
but written in expl3. The goal was just an exercise in using expl3, but if you find a bug
feel free to open an issue or PR.

## Sample document
```tex
\documentclass{article}
\usepackage{amssymb,amsmath}
\usepackage{key-theorems}
\usepackage{hyperref}

\newkeytheoremstyle{mystyle}{
    % <insert options>
    }
\newkeytheorem{theorem}[
    style=mystyle,
    parent=section
    ]
\newkeytheorem{remark}[
    style=remark,
    numbered=no
    ]

\begin{document}

\section{Some theorems}

\begin{theorem}[
    note=Strong Bertini over $\mathbb{C}$,
    label=strongbertini
    ]
Let $X$ be a smooth complex variety and let $\mathfrak{D}$ be a positive dimensional linear system on $X$.
Then the general element of $\mathfrak{D}$ is smooth away from the base locus $B_{\mathfrak{D}}$. That is, the set
    \[\{H\in\mathfrak{D}\mid D_H \text{ is smooth away from } B_{\mathfrak{D}}\}\]
is a Zariski dense open subset of $\mathfrak{D}$.
\end{theorem}

\begin{remark}
In fact, \autoref{strongbertini} holds over any algebraically closed field of characteristic zero.
\end{remark}

\begin{theorem}[Bertini over any field]
Let $X\subset\mathbb{P}_k^n$ be a smooth projective variety over a field $k$. Then the set of hyperplanes
$H\subseteq\mathbb{P}_k^n$ such that $X\cap H$ is smooth is a Zariski dense open subset of $(\mathbb{P}_k^n)^*$.
\end{theorem}

\listofkeytheorems

%%% compare with
%\listofkeytheorems[ignoreall,show=theorem]
%\listofkeytheorems[swapnumber]
%\listofkeytheorems[title=BLUB]
%\listofkeytheorems[print-body] % needs 'store-all' package option

\end{document}
```

## Documentation
There is a list of commands and keys offered by the package
[here](https://github.com/mbertucci47/key-theorems/blob/main/doc/key-theorems-doc.pdf).
More of a reference document than documentation.

## Differences with `thmtools`

Most of the code is a direct translation from `thmtools`
but a few things are changed:
- The only backend supported is `amsthm`, and it is loaded by the package. As I understand it,
  [`ntheorem`](https://www.ctan.org/pkg/ntheorem) has quite a few bugs and no active development, and
  if you're just using the kernel `\newtheorem` then you don't need a package like this.
- Theorem hooks use the kernel hooks. Thus unlike the `prefoot` and `postfoot` hooks of
  `thmtools`, all code chunks are added to hooks in the order declared (unless given a
  distinct label; then the chunks are reversed). The order of generic/specific hooks
  remains the same. Furthermore, the command for adding to theorem hooks is `\addtotheoremhook{<hook name>}{<theorem name>}`.
- `thmtools` changes some style settings to be default that are different from
  amsthm defaults, but only if a custom style is called. Example:
  ```tex
  \declaretheoremstyle[spaceabove=20pt]{mythmsty}
  \declaretheorem[style=mythmsty]{mythm}
  ```
  
  is different from just
  
  ```tex
  \declaretheorem{mythm}
  ```
  
  as in the former, `bodyfont` is `\normalfont`, not the default `\itshape`.
  This package keeps the defaults unless a key is specifically given.
- The `restate=` code uses sequences from l3seq. What does this mean for speed/robustness?
  I have no idea. Also this is only implemented as a key, no `restatable` environment except with `thmtools-compat`.
- Rather than `restate=foo` defining a command `\foo*`, theorems are retrieved with `\getkeytheorem{foo}`.
  For just the theorem body, use `\getkeytheorem[body]{foo}`.
- The `shaded` and `thmbox` keys are only available with the package option `thmtools-compat`,
  and they are emulated with [`tcolorbox`](https://www.ctan.org/pkg/tcolorbox). Instead there's
  an interface to `tcolorbox` with the `tcolorbox={<options>}` and `tcolorbox-no-titlebar={<options>}` key.
  The `mdframed` key is not implemented.
- You can reuse styles with the `inherit-style` key.
- With `thmtools`, the command `\newtheorem{<envname>}{<heading>}` is changed to behave like
  `\declaretheorem[name=<heading>]{<envname>}`. This is not the default here. Instead either
  only use the new interface or load the package with option `overload`.
- All new keys (see doc for descriptions):
  - load time: `overload`, `thmtools-compat`, `store-all`
  - global (in `\keytheoremset`): `restate-counters`, `continues-code`, `qed-symbol`
  - keys for theorem envs: `note` (alias `name`), `short-note`, `continues*`, `store` (alias `restate`)
  - declaring theorems (in `\newkeytheorem`): `tcolorbox`, `tcolorbox-no-titlebar`
  - declaring styles (in `\newkeytheoremstyle`): `inherit-style`
  - list of theorems (in `\keytheoremlistset` and `\listofkeytheorems`): `title-code`, `no-title`, `note-code`, `print-body`

## thmtools issues resolved by key-theorems
Issues from the thmtools [github page](https://github.com/muzimuzhi/thmtools),
with corresponding key-theorems code that shows the issue resolved.
### [\listoftheorems: filter out restated ones #7](https://github.com/muzimuzhi/thmtools/issues/7)
key-theorems does not list restated theorems in the `\listofkeytheorems`.
```tex
\documentclass{article}

\usepackage{key-theorems}

\newkeytheorem{theorem}

\begin{document}

\begin{theorem}[store=foo]
text
\end{theorem}

\getkeytheorem{foo}

\listofkeytheorems

\end{document}
```

### [\listoftheorems: hide title #8](https://github.com/muzimuzhi/thmtools/issues/8)
key-theorems provides the `no-title` key.
```tex
\documentclass{article}

\usepackage{key-theorems}

\newkeytheorem{axiom}
\newkeytheorem{theorem}

\begin{document}

\listofkeytheorems[ignoreall,show=axiom]
\listofkeytheorems[ignoreall,show=theorem,no-title]
     
\begin{axiom}
some axiom
\end{axiom}

\begin{theorem}
some theorem
\end{theorem}

\begin{axiom}
another axiom
\end{axiom}

\begin{theorem}
another theorem
\end{theorem}

\end{document}
```

### [Case in point: the tcolorbox key #9](https://github.com/muzimuzhi/thmtools/issues/9)
key-theorems provides the `tcolorbox` and `tcolorbox-no-titlebar` keys.
```tex
\documentclass{article}

\usepackage{key-theorems}

\newkeytheorem{theorem}[
  tcolorbox={
    colback=blue!10,
    colframe=blue!75
    }
  ]
\newkeytheorem{corollary}[
  tcolorbox-no-titlebar={colback=red!10}
  ]

\begin{document}

\begin{theorem}
some theorem
\end{theorem}

\begin{corollary}
some corollary
\end{corollary}

\end{document}
```

### [thm-listof: document \thmtformatoptarg, new list option? #14](https://github.com/muzimuzhi/thmtools/issues/14)
key-theorems provides the `note-code` key for `\listofkeytheorems` and
`\keytheoremlistset`.
```tex
\documentclass{article}

\usepackage{key-theorems}

\newkeytheorem{theorem}

\begin{document}

\begin{theorem}[my heading]
some theorem
\end{theorem}

\listofkeytheorems[note-code={ [\textit{#1}]}]

\end{document}
```

### [Style inheritance in thmtools #15](https://github.com/muzimuzhi/thmtools/issues/15)
key-theorems provides the `inherit-style` key.
```tex
\documentclass{article}

\usepackage{key-theorems}

\newkeytheoremstyle{mysty1}{notefont=\itshape}
\newkeytheoremstyle{mysty2}{inherit-style=mysty1,bodyfont=\normalfont}
\newkeytheorem{theorem}[style=mysty1]
\newkeytheorem{corollary}[style=mysty2]

\begin{document}

\begin{theorem}[my heading]
some theorem
\end{theorem}

\begin{corollary}[my heading]
some corollary
\end{corollary}

\end{document}
```

### [If wrapped in tcolorbox, beginning \parskip is always inserted #16](https://github.com/muzimuzhi/thmtools/issues/16)
This is a tcolorbox issue with lists (amsthm theorems are internally lists).
Avoided with the `tcolorbox` and `tcolorbox-no-titlebar` keys.
```tex
\documentclass{article}

\usepackage{key-theorems}

\newkeytheorem{theorem}[tcolorbox-no-titlebar]

\parskip=10pt

\begin{document}
\tcbset{parbox=false}

\begin{theorem}
some theorem
\end{theorem}

\end{document}
```

### [Restate without theorem headings (head + note) #19](https://github.com/muzimuzhi/thmtools/issues/19)
Use `\getkeytheorem[body]{foo}`.
```tex
\documentclass{article}
\usepackage{key-theorems}

\newkeytheorem{theorem}

\begin{document}

\begin{theorem}[store=hello]
Hello!
\end{theorem}

\getkeytheorem{hello}

\section{\getkeytheorem[body]{hello}}

\end{document}
```

### [Option clash: numbered=no and thmbox #25](https://github.com/muzimuzhi/thmtools/issues/25)
Fixed with key-theorem's implementation of the `thmbox` key with `thmtools-compat`.
```tex
\documentclass{article}

\usepackage[thmtools-compat]{key-theorems}

\declaretheorem[numbered=no, name=TheoremA,         ]{mytheo1}
\declaretheorem[             name=TheoremB, thmbox=M]{mytheo2}
\declaretheorem[numbered=no, name=TheoremC, thmbox=M]{mytheo3}

\begin{document}

\begin{mytheo1}
    test1
\end{mytheo1}

\begin{mytheo2}
    test2
\end{mytheo2}

\begin{mytheo3}
    test3
\end{mytheo3}

\end{document}
```

### [Skip the title of the theorem but leave the []. #30](https://github.com/muzimuzhi/thmtools/issues/30)
Fixed in key-theorems.
```tex
\documentclass{article}
\usepackage{key-theorems}

\newkeytheorem{Teorema}[name=Theorem]

\begin{document}

\begin{Teorema}
  body
\end{Teorema}

\begin{Teorema}[]
  body
\end{Teorema}

\begin{Teorema}[title]
  body
\end{Teorema}

\end{document}
```

### [too much space with restatable #40](https://github.com/muzimuzhi/thmtools/issues/40)
Fixed in key-theorems.
```tex
\documentclass{article}
\usepackage{key-theorems}
\usepackage{kantlipsum}

\newkeytheorem{theorem}

\begin{document}

\begin{theorem}
\kant[2][1]
\end{theorem}

\begin{theorem}[store=foo]
\kant[1][2]
\end{theorem}

\begin{theorem}
\kant[2][1]
\end{theorem}

\getkeytheorem{foo}

\end{document}
```

### [restate key with no name leads to (,) in restated title #41](https://github.com/muzimuzhi/thmtools/issues/41)
Fixed in key-theorems.
```tex
\documentclass{article}
\usepackage{key-theorems}
\usepackage{kantlipsum}

\newkeytheorem{theorem}

\begin{document}

\begin{theorem}[restate=foo]
\kant[1][2]
\end{theorem}

\getkeytheorem{foo}

\begin{theorem}[restate=foobar,name=Heading]
\kant[2][1]
\end{theorem}

\getkeytheorem{foobar}

\end{document}
```

### [prefoot and postfoot hooks called twice with restate key #54](https://github.com/muzimuzhi/thmtools/issues/54)
Fixed in key-theorems.
```tex
\documentclass{article}
\usepackage{key-theorems}

\newkeytheorem{theorem}

\addtotheoremhook{prehead}{PREHEAD}
\addtotheoremhook{posthead}{POSTHEAD}
\addtotheoremhook{prefoot}{PREFOOT}
\addtotheoremhook{postfoot}{POSTFOOT}

\begin{document}

\begin{theorem}
body text
\end{theorem}

\begin{theorem}[store=foo]
body text
\end{theorem}

\end{document}
```

### [restate key incompatible with beamer #58](https://github.com/muzimuzhi/thmtools/issues/58)
```tex
\documentclass{beamer}
\setbeamertemplate{theorems}[numbered]
\usepackage{key-theorems}

\newkeytheorem{MyTheorem}

\begin{document}

\begin{frame}
\begin{MyTheorem}[restate=foo]
text
\end{MyTheorem}

\getkeytheorem{foo}
\end{frame}

\end{document}
```

## Things to do

- Clean up the code. Things are out of order, poorly named, etc.
- Add more error messages.
- `thmtools` features not yet implemented
    - labels pointing to restated theorem, not original
    - [`beamer`](https://ctan.org/pkg/beamer) support
    - certainly more
- For a complete list, see the bottom of [`key-theorems.sty`](https://github.com/mbertucci47/key-theorems/blob/main/key-theorems.sty)
