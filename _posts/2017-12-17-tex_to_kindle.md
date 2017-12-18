---
layout: post
title: "No Printer While Traveling Trick"
categories: [Workflow]
tags: [latex]
---
I hate reading on screens. It strains my eyes, I retain less, and I miss things. I particularly hate trying to edit on screens because that just exacerbates these problems. So I buy the paper copy of the book whenever possible and print important things I need to read.

But books are heavy to carry and expensive to buy in South America so when I'm traveling there I borrow an ereader. That way I can still unwind with some literature at the end of the day. This past semester was the first trip I've tried this, a bit to my surprise, the experience wasn't terrible. The normal problems I associate with a screen weren't really present with the device. Now, it's not 100% the same and I'm not about to give up my library, but for travel it'll do.

What about editing? All the problems of reading on screens are magnified in editing and the stakes are even higher because the things I have to edit are, essentially, what get me paid. So this got my thinking: if an ereader makes reading less terrible, could it also make editing better?

This post is about how to do this (described below), but let me answer the question first. Yes, and no. Back in middle school English class you probably learned that the "writing process" involves a series of steps, two of which are revising and editing. You've probably realized since then that's a bit of a gross oversimplification/complete lie. But it's a nice model for describing my experiences. The ereader made it easier to do the "editing" --- proofreading for grammar and spelling; catching small errors; and listening to the words on the page. I did not find it great for "revising" --- major revisions, structural changes, and the like. The small screen makes it tough to really get a feel for the whole piece of writing, there's no easy way to jump around in the document like I would on paper, and, most importantly, marking it up was basically out the window.

Still, reading to correct the small mistakes is an important step and for that, the ereader was a big win. So here's how to make that happen if you find yourself in a similar situation (spoiler alert, it's really easy). You'll need a recent version of pandoc and, if you're using a Kindle, some additional tools from Amazon.

# The Technical Solution
## Installing Pandoc
`brew install pandoc pandoc-citeproc`

You'll need pandoc-citeproc for dealing with references.

## Installing Amazon tools
Download the KindleGen tools from [here](https://www.amazon.com/gp/feature.html?docId=1000765211). I'd recommend moving the KindleGen file (and the legal documents) somewhere convenient and then adding that location to your `PATH`. In my case, it went into ~/.config/kindlegen and I added `PATH="$PATH:~/.config/kindlegen"` to the end of my `.bash_profile`.

## Converting for an Ebook
Here are all the steps I would take for converting from \TeX file to the Kindle format and coyping it onto the device connected via USB:

```bash
cd my_working_directory
pandoc -s my_doc.tex --bibliography=/full/path/to/bib_file.bib -o my_doc.epub
kindlegen my_doc.epub
cp my_doc.mobi /Volumes/my_kindle/Documents/
rm my_doc.epub my_doc.mobi
```
# The Bottom Line
Is this useful? Maybe. Maybe not. I think it works well for the limited situation I describe. It could also be useful if you want to see your dissertation on an ereader. I also noticed that the most recent version of APSR gives an option to download ereader versions of the articles. So maybe you having something in the works and want to double-check how it might look in that format.

Admittedly, these are some fairly narrow use-cases. Maybe you'll find something better to do with this information!
