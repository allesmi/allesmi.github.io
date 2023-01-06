+++
title = "Create documents for your tabletop RPG campaign in LaTeX"
date = "2020-10-06T20:37:10+02:00"
tags = ["latex"]
+++

I wanted to create an introductory document for a [Dungeons and Dragons](https://dnd.wizards.com/) campaign that we want to start soon.
It should give my players an idea what to expect during the adventure and in what universe their characters will act.
Using LaTeX to create a beautifully typeset PDF seems like an obvious choice.

I use [git](https://git-scm.com/) to track changes over time, [LaTeX](https://www.latex-project.org/) for typesetting, and the [DND-5e-LaTeX-Template](https://github.com/rpgtex/DND-5e-LaTeX-Template) to make the document visually interesting as well.

Firstly, create a new directory for your adventure and initialize the git repository.

```bash
mkdir my-campaign
cd my-campaign
git init
```

Apart from the introduction to the campaign, I expect that other written content will follow.
To keep everything tidy, I will create two directories.

```bash
mkdir introduction
mkdir vendor
```

The `introduction` directory will hold our LaTeX files.
All third-party content will be stored in `vendor`.

Let's add the LaTeX template into the `vendor` directory as a git submodule.

```bash
git submodule add https://github.com/rpgtex/DND-5e-LaTeX-Template.git vendor/DND-5e-LaTeX-Template
git add .gitmodules
git commit -m "Add DND-5e-LaTeX-Template as submodule"
```

Change into the `introduction` folder and create the files `introduction.tex` and `latexmkrc`.

```bash
cd introduction
touch introduction.tex
touch latexmkrc
```

Since the template we added to our project earlier is not in the search path of `latexmk`, we have to add it manually.
We could set the environment variable `TEXINPUTS` but that would remove all other search paths.
We can however add a directory to `TEXINPUTS` in a config file for `latexmk`.

Open the `latexmkrc` file and add the following line.

```
ensure_path( 'TEXINPUTS', '../vendor/DND-5e-LaTeX-Template//' );
```

Now add content to your LaTeX file, like for example:

```tex
\documentclass[10pt,twoside,twocolumn,openany]{dndbook}

\usepackage[english]{babel}
\usepackage[utf8]{inputenc}

\title{My Campaign\\
\large Introduction}
\author{Author's Name}
\date{\today}

\begin{document}

\frontmatter
\maketitle
\tableofcontents

\mainmatter%
\part{Introduction}
% ...

\end{document}
```

Finally, to see changes to your LaTeX file side by side with the resulting pdf use `latexmk`.

```bash
latexmk -pvc -pdf introduction.tex
```

When everything looks good commit your changes.

```bash
git add latexmkrc
git add introduction.tex
git commit -m "Start with introduction"
```

The example file given in the DND-5e-LaTeX-Template repository will show how to continue from here on.