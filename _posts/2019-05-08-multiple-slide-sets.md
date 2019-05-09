---
layout: post
title: "Preparing Slides for Teaching"
categories: [teaching]
tags: [latex,makefile,code]
---
I know there are plenty of great posts out there already about preparing slides using some combination or markdown/org-mode/LaTeX with a Makefile, but I found that no single one covered the precise combination of things I want to do. Also, some of the fundamental tools (coughPANDOCcough) have changed in breaking ways. So here's my take on combining a bunch of awesome plain text tools together for preparing lecture slides. 

# My (weird?) situation
Here's what I wanted to be able to do: write slides in org-mode[^orgvsmkdown] and then automatically convert them into four sets of pdf slides: one set to display in the classroom that includes things like `\pause`, a second "handout" set to distribute to students, a third "handout" set for student but with space next to the slides so they could print them and take notes next to the material, and a fourth instructor set that includes notes I've written myself. 

I also wanted to be able to use a customized template for my beamer slides because I became obsessed with the idea that I should use official school colors for these things. A complicating factor here was that my template came in the form of a `.cls` file and not a `.sty`, causing pandoc to freak out.

And to complicate things further, I want to be able to include a set of "current events" slides that will appear only on the display set of slides. I update these until just about the last minute, but have no need to include them in my set of notes and no desire to include them in the student version. 

Oh, and the solution needed to work on both Linux and OSX.

My first attempt at this failed. It involve an unwieldy Makefile that reran things unnecessarily and did not create four distinct sets of slides. Then I switched to Emacs and my second attempt also failed. It involved creating my slides, then creating two near duplicates of the org-mode file and trying to export withing Emacs. Automation was out. As was my custom template and a good part of my sanity. 

This semester things are different! I've solved my problems on attempt number 3.

# Attempt Number 3
There are really only 4 tools required here: pandoc, GNU make, \LaTeX, and your favorite text editor.

## The org-mode header
The first trick is to set things up correctly in the org-mode document header: 

{% raw %}
```org-mode
#+TITLE:     Some Title
#+AUTHOR:    Me!
#+DATE:      Today's Date
#+startup: beamer
#+LATEX_CLASS: beamer
#+LATEX_COMPILER: xelatex
#+LANGUAGE: en
#+OPTIONS: toc:nil H:2
#+LATEX_CLASS_OPTIONS: [aspectratio=1610]
#+LATEX_HEADER: \setbeamertemplate{navigation symbols}{}%remove navigation symbols
$0
#+latex_header: \graphicspath{{/OSX/path/to/project/}{/Linux/path/to/project/}}
```
{% endraw %}

Most of that should be pretty self-explanatory, I think. I don't want a table of contents in the slides and the second level heading should define a new slide. 

The final line is really helpful. Pandoc's beamer template will automatically load the `graphicx` package. `\graphicspath` lets you specify a list of possible paths to check. That way in the slides a simple `\includegraphics{}` statement can just use a relative path. 

## The Templates
Now come the templates. I have two of them, stored in a "templates" directory alongside the slide files. They're pretty simple. Here's a snippet of "slide_notes_header.tex" to give you an idea:

```latex
\usepackage{pgfpages}
\setbeameroption{show notes}
\setbeamertemplate{note page}[plain]

% stick slide number on each note: notice division by two!

\usepackage{calc}
\newcounter{curslide}
\addtobeamertemplate{note page}{%
    \setcounter{curslide}{\thepage / 2}%
    SLIDE \thecurslide \par}{}

% force notes on each slide
% http://tex.stackexchange.com/questions/11708/run-macro-on-each-frame-in-beamer

\usetheme{metropolis}
\usecolortheme{beaver}
\definecolor {utorange}   {RGB} {191, 87, 0}
\definecolor{foocolor}{RGB}{26, 146, 10} % ugly green to use for figuring out where each color appears in the metropolis frames
\setbeamercolor {normal text} {bg=white, fg=black}
\setbeamercolor {structure} {fg=utorange}

```

The other template is basically the same, but without the notes stuff. Note that in both, I call `\usetheme{metropolis}` to specify the theme at this point, and not in the org-mode header. 

## The Makefile
So now we can put this together and save all of the work:

```makefile
.PHONY: all
all: slides 

current_dir = $(shell pwd)
lsd = lecture_notes
lsds = $(lsd)/student
lsdp = $(lsd)/professor
lsdd = $(lsd)/display
lsdn = $(lsd)/student_printable
met_header=$(lsd)/templates/metropolis_header.tex
snh = $(lsd)/templates/student_slide_notes_header.tex

# slides
slides: student_slides prof_slides display_slides student_notes_slides 

student_slides: $(lsds)/1_intro.pdf
student_notes_slides: $(lsdn)/1_intro.pdf
prof_slides: $(lsdp)/1_intro.pdf
display_slides: $(lsdd)/1_intro.pdf

# student slides
$(lsds)/%.pdf: lecture_notes/%.org $(met_header)
	pandoc $< -f org+smart -t beamer -o $@ --pdf-engine xelatex --variable classoption=handout  --variable aspectratio=1610 -H $(met_header)

# professor slides
$(lsdp)/%.pdf: lecture_notes/%.org lecture_notes/templates/slide_notes_header.tex
	pandoc $< -f org+smart -t beamer --pdf-engine xelatex -H $(lsd)/templates/slide_notes_header.tex --variable classoption=handout --variable aspectratio=1610 -o $@

# display slides
$(lsdd)/%.pdf: lecture_notes/%.org $(met_header)
	awk '//; /Gov310\/\}\}/{while(getline line<"lecture_notes/current_events.org"){print line}}' $< > lecture_notes/tmp.org; \
	pandoc lecture_notes/tmp.org -f org+smart -t beamer --pdf-engine=xelatex -o $@ -H $(met_header); \
	rm lecture_notes/tmp.org

# student printable slides
$(lsdn)/%.pdf: lecture_notes/%.org lecture_notes/templates/slide_notes_header.tex
	cat $< | perl -pe 's/\\note.*?\{.*?\}/\\note\{\}\n/' > lecture_notes/tmp2.org; \
	pandoc lecture_notes/tmp2.org -f org+smart -t beamer --pdf-engine xelatex --variable classoption=handout -H lecture_notes/templates/slide_notes_header.tex --variable aspectratio=1610 -o $@; \
	rm lecture_notes/tmp2.org


```

There are a few crucial bits here. First, it uses the new pandoc convention where you specify modifiers with the `+` syntax. In this case, I want to process plain vertical quotation marks as "smart" (sloping or wrapping in the correct direction on each side). Second, the `-H` flag allows me to set the header. This doesn't replace the entire pandoc template header, but rather supplements it.[^pandoctemplate] Third, I can set \LaTeX class options with `==variable classoption=handout`. Doing this overwrites the classoption(s) set from within the org-mode document, though, so I have to also re-specify that aspectratio in two cases. 

The display slides are more complicated because that's where I bring in the additional current events slides. I'm using `awk` to copy the slides (contained in a separate file) to just after the top matter in the slide file ends. Then everything gets processed as normal and cleaned up again at the end. Note that I *don't* include the current_events.org file as a prerequisite. If I were to do this, I'd end up remaking *all* of the display slides every time I updated that, which I don't want. This does mean that I have to force `make` to do what I want. I usually just use `touch` to update the timestamp on the slides I want to process. I know this breaks some of the automation, so if anybody has a better solution, I'm all ears. 

The full Makefile does some more stuff, too (syllabus, assignments, exams). The only clunky thing that I don't like is that I have to specify the pdfs four times, essentially. And those get updated as I build out the course during the semester. If anybody knows a way to make this even easier, I'd love to know what it is!

Another quick caveat: I like to run multiple jobs at once with `make` (specified with the `-j` flag). Given the hacky way I've set up processing the display and student printable slides this fails if you try to update multiple slides at once (i.e. two different lectures, which might happen if I'm preparing slides for a week or more in advance). My solution is to make sure I only do one at a time or run only one job at once if absolutely necessary. 

# Other Tricks
Here are a few other things I figured out for creating beamer presentations from org-mode files. 

1. There are probably things you can do in pure LaTeX that you cannot do in org-mode. This was a painful realization, but I've also accepted that those things are relatively rare and the benefits of org-mode outweigh those downsides. 
2. Two column-setup: Sometimes I like to have slides that have two columns. Usually it's one of text and one image. Here's how I do that:
```org-mode
#+begin_columns
#+attr_html: :width 50%
#+begin_column
#+attr_latex: :width=\textwidth
\includegraphics{lecture_notes/images/bonnie_clyde.jpg}
#+end_column
#+attr_html: :width 50%
#+begin_column
- Game designed to illustrate a collective action problem
\pause
- Two robbers caught -- let's call them Bonnie and Clyde
\pause
- Each offered a deal: No time if they confess and their partner will serve 10 years
\pause
- But if they both confess, then they will each serve 5 years
\note[item]{Collection action problem: any situation in which individuals are better free-riding and enjoying a public good than contributing to its provision}
\note[item]{Public good: benefit for a group that any member can enjoy without paying}
#+end_column
#+end_columns
```
3. As you'll *note* above, the way I embed notes within the slides for myself (that will appear on the instructor set of slides) is with the `\note[item]{}` command. This is what's defined in the lecture_notes_header.tex. The `[item]` specification is important if you have more than one note per slide - it creates a numbered list. Otherwise your lines run together.

[^orgvsmkdown]: Originally I wanted to write them in markdown, but I've graduated to Emacs, so my whole life is org-mode now, obviously. 

[^pandoctemplate]: If you want to use your own template entirely, that's possible. Check the pandoc documentation. 
