# Multi-file LaTeX Projects

In large LaTeX documents one usually has several `.tex` files, one for each
chapter or section, and then they are joined together to generate a single
output. This helps to keep everything organized and makes easier to debug
the document, but as the document gets larger the compilation takes longer.
This can be frustrating since one is usually only interested in working in
a particular file each time.

The natural approach to overcome this is to compile each file separately.
There are two main packages that allow compilation of single files
in a multi-file project.
The choice you make depends on what you need.

- With the `subfiles` package
you can compile every subfile independently
and each subfile will automatically use the preamble in the main file.
- With the `standalone` package every subfile works as
an independent file, subfiles can be latter joined in a main document
that will pull the preambles from each one of them.
Especially useful if you need to reuse the same file in more than one document,
a tikz picture is a good example of this.
The package `standalone` provides the same functionality as `subfiles`
and is more flexible, but the syntax is more complex and prone to
errors. The main difference is that each subfile has its own preamble.

## Management in a large project

In large projects, such as books, keeping parts of your document 
in several `.tex` files makes the task of correcting errors 
and making further changes easier. It's simpler to locate a 
specific word or element in a short file.

### Inputting and including files

The standard tools to insert a LaTeX file into another are `\input` and `\include`.

#### `input` command

`\input{filename}`

Use this command in the document body to insert the contents of 
another file named `filename.tex`;
this file should not contain any LaTeX preamble code 
(i.e. no `\documentclass`, `\begin{document}` or `\end{document}`).
LaTeX won't start a new page before processing the material in 
`filename.tex`. `\input` allows you to nest `\input` commands,
in files that are already being inputted by the root file.

#### `include` command

`\input{filename}`

Use this command in the document body to insert the contents of
another file named `filename.tex`;
again this file should not contain any LaTeX preamble.
LaTeX will start a new page before processing the material input 
from `filename.tex`. Make sure not to include the extension `.tex`
in the `filename`, as this will stop the file from being input 
(the extension can optionally be included with input and import).
It is not possible to nest include commands. 
Each file that gets `\included` has its own .aux file storing information
of created labels and contents for the table of contents,
list of figures, etc. You can use `\includeonly` with 
a comma separated list of file names (make sure that there are 
no leading or trailing spaces).
If you do this LaTeX will only process the files contained in that list.
This can be used to enhance compilation speed if you're only working 
on a small part of a bigger document.
Page numbers and cross references will however still work,
as the `.aux` files of left out files will still be processed.

### Preamble in a separate file

If the preamble of your document has many user-defined commands 
or term definitions for the glossary, you can put it in a separate file.
The right way is to create a custom package,
which is a file with the `.sty` extension.
Let's see an example:
```latex
\ProvidesPackage{example}

\usepackage{amsmath}
\usepackage{amsfonts}
\usepackage{amssymb}
\usepackage[latin1]{inputenc}
\usepackage[spanish, english]{babel}
\usepackage{graphicx}
\usepackage{blindtext}
\usepackage{textcomp}
\usepackage{pgfplots}

\pgfplotsset{width=10cm,compat=1.9}

%Header styles
\usepackage{fancyhdr}
\setlength{\headheight}{15pt}
\pagestyle{fancy}
\renewcommand{\chaptermark}[1]{\markboth{#1}{}}
\renewcommand{\sectionmark}[1]{\markright{#1}{}}
\fancyhf{}
\fancyhead[LE,RO]{\thepage}
\fancyhead[RE]{\textbf{\textit{\nouppercase{\leftmark}}}}
\fancyhead[LO]{\textbf{\textit{\nouppercase{\rightmark}}}}
\fancypagestyle{plain}{ %
\fancyhf{} % remove everything
\renewcommand{\headrulewidth}{0pt} % remove lines as well
\renewcommand{\footrulewidth}{0pt}}

%makes available the commands \proof, \qedsymbol and \theoremstyle
\usepackage{amsthm}

%Ruler
\newcommand{\HRule}{\rule{\linewidth}{0.5mm}}

%Lemma definition and lemma counter
\newtheorem{lemma}{Lemma}[section]

%Definition counter
\theoremstyle{definition}
\newtheorem{definition}{Definition}[section]

%Corolary counter
\newtheorem{corolary}{Corolary}[section]

%Commands for naturals, integers, topology, hull, Ball, Disc, Dimension, boundary and a few more
\newcommand{\E}{{\mathcal{E}}}
\newcommand{\F}{{\mathcal{F}}}
...

%Example environment
\theoremstyle{remark}
\newtheorem{examle}{Example}

%Example counter
\newcommand{\reiniciar}{\setcounter{example}{0}}
```

All the commands in this file could have been put in the preamble,
but the main file would have become confusing because of
this large amount of code, and to locate the actual body of
the document in such large file would be a cumbersome task.

This file could also be put in a normal `.tex` file and imported with
the command `import`, but a `.sty` file prevents possible errors
if the file is accidentally imported more than once.

Notice that the first line of the example is `\ProvidesPackage{example}`,
this means that we have to import this package as example in the main file,
i.e. with the command `\usepackage{example}`

Note: A `.sty` file is far more flexible,
it can be used to define your own macros and optional parameters can be passed.

### Using the `import` package

As mentioned above The standard tools to insert a LATEX file 
into another are `\input` and `\include`,
but these are prone to errors if nested file importing is needed.
For this reason you may want to consider the package `import`.

Below is an example of a book whose sections and user-defined commands are stored in separate files,
and where the image files for each chapter are stored in separate folders along
with the `.tex` file for each chapter.
```latex
\documentclass[a4paper,11pt]{book}
\usepackage{import}
\usepackage{example}

\usepackage{makeidx}
\makeindex

\begin{document}

\frontmatter
\import{./}{title.tex}

\clearpage
\thispagestyle{empty}

\tableofcontents

\mainmatter
\chapter{First chapter}
\import{sections/}{section1-1.tex}
\import{sections/}{section1-2.tex}

\chapter{Additional chapter}
\import{sections/}{section2-1.tex}

\chapter{Last chapter}
\import{sections/}{section3-1.tex}

\backmatter

\import{./}{bibliography.tex}

\end{document}
```
As you see, the example is a book with three chapters 
and several sections in a neat main file that pulls external files 
to generate the final document.
The command `\frontmatter` in the book document class is used for the first
pages of the document, the page numbering style is set to Roman numerals by this
command; the command `\mainmatter` resets the page numbering and changes the
style to Arabic, `\backmatter` disables the chapter numbering 
(suitable for the bibliography and appendices).
```latex
\chapter{First chapter}
\import{sections/}{section1-1.tex}
\import{sections/}{section1-2.tex}
```
First, add this line to the preamble of your document:
```latex
\usepackage{import}
```
Then use `\import{ }{ }`.
The first parameter inside braces is the directory where
the file is located, it can be relative to the current working directory or
absolute. The second parameter is the name of the file to be imported.

There is also available the command `\subimport` that has the same syntax,
but if used in one of the files that are imported in the main file,
the path will be relative to that sub-file.
For instance, below is the contents of the file "section1-1.tex" that was imported in the previous example:
```latex
\section{First section}

Below is a simple 3d plot

\begin{figure}[h]
\centering
\subimport{img/}{plot1.tex}
\caption{Caption}
\label{fig:my_label}
\end{figure}

[...]
```
As you see, this file imports a pgf plot file called "plot1.tex" that creates a 3d plot. This file is imported by
```latex
\subimport{img/}{plot1.tex}
```
from the "img" folder, contained in the folder "sections".

If `\import` were used instead, the path `img/` would be relative to the main file, instead of the folder "sections" where "section1-1.tex" is saved.
