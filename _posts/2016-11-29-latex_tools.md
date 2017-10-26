---
layout: post
title: "Some Random LaTeX Tools"
categories: [Code]
tags: [latex, workflow]
---
LaTeX is a really wonderful tool for authoring documents in the academic world, but new users often look at it and feel like any gains that might be associated with it are more than offset by loss of many of the conveniences of a word processor. While I think that the benefits far outweigh any costs, I'm only going to argue here that you don't actually lose all of the features you might think you lose.

# Spell check
I'll admit it: I'm terrible at spelling. And even though spell check remains a [relatively dumb tool](http://cavern.uark.edu/~arnold/Other/ZarOde.pdf) and requires the author to actually think about replacements, the utility of seeing a little red squiggle to alert me to the fact that I have written something that may not be a real word is pretty high. When I first switched to using LaTeX, I lost that ability and it hurt. But then I realized there are a number of good options to get it back! So here are some options ordered (in my opinion) from worst to best:
  1. Run the file through Excalibur, the small additional program that probably came with your TeX distribution. It's a little outdated (it thinks pretty much every internet-related word is misspelled), but it'll get the job done. Just be sure to save your file after you make changes (and then re-typeset it)!
  2. Use a TeX program that has spell-check built in. TeXshop does, and I'm pretty sure that LyX does as well (but if you're going to use LaTeX, LyX defeats a lot of the purpose, so don't go that route).
  3. Use a text editor that supports spellcheck. I know that TextWrangler and Atom both do it (Atom requires one tiny bit of setup) and I imagine that Sublime Text does as well. (Yes, Emacs can do it as well. Of course.)

# Word Count
Word count is that feature that seemed really important in middle school when writing was super painful and getting to 500 words seemed like an unreachable target. Then it seemed really irrelevant for a long time. And then grad school and journal word limits came along. Luckily, this is also easily solved using the texcount tool that is bundled with LaTeX installations.

If you're on a Mac, open the terminal application and use the `cd` command to navigate to the folder where your .tex file is located.[^1] For example, if it's in a folder called "Paper" that's within a folder called "Stats" within "Documents", then you would type `cd ~/Documents/Stats/Paper` and hit enter. Note that the terminal will help you autocomplete paths, so if you started typing "cd ~/Docu" and then hit tab, it would automatically add on "ments/" (assuming you didn't have any other folders that start with "Docu" within that directory).

Now that your working directory is set to the location of your tex document, just type `texcount <file>` where you replace "<file>" with the name of your tex document (including the .tex extension). There are a number of options you can run with the command. See [this](http://app.uio.no/ifi/texcount/howto.html) website for full details. As an example, you might run `texcount -incbib <file>` to also count words in the bibliography.

I'm sure that there are word count plugins for the popular text editors out there, but I don't have any experience with those. Again, this is something that I don't worry about enough to want to monitor closely, but is useful at times to make sure that I'm within some specified guidelines.

[^1]: If you're on a PC, I can't give you advice about what shell application to use or how to properly write the path to your directory, but the procedure described here should be the same, at least.
