---
layout: post
title: "Collaborative Academic Writing"
categories: [Workflow]
tags: [latex]
---
Academia has a reputation for being a [lonely profession](https://www.amazon.com/Stoner-York-Review-Books-Classics/dp/1590171993) and much of that is deserved. But there are also lots of collaborative opportunities in research. Somewhat to my surprise, these collaborative projects have become some of my favorite parts of my (brief) academic career. Working together entails all kinds of additional considerations from how to divide up the work to how to manage disputes when you and your coauthors disagree. And then there's the part of figuring out how to actually *work* together - as in how do multiple people run the same analysis and then write in the same document. There are a number of different solutions for this problem and there's probably no single "right" answer. This post will share some thoughts on what has and has not worked for me, based on a few recent collaborative projects.

First, here are some things I really want when I'm collaborating with other authors:
  * The ability for all authors to work on a document at the same time
  * The ability to easily reconcile different versions of a document
  * The ability to see differences between documents and easily merge those changes
  * Backups of the files so that a single author can't destroy the entire project when their laptop crashes/their account is hacked/total incompetence
  * A single version of the document (note: this is different from a single copy)

And here are some things I really *don't* want when I'm collaborating with other authors:
  * To be locked out of a document because another author is editing it (or was editing it and then stepped away from the computer)
  * To have one author's version of a document overwrite another's without incorporating changes from both
  * To have to spend time going line by line to reconcile two documents
  * To have the entire project disappear because an author's toddler happened to hit the right combination of keys to permanently delete everything
  * Multiple versions with increasingly meaningless names (e.g. manuscript_v1, manuscript_v1a, manuscript_v1a_ba_edits,...)
  * To only be able to edit the documents while I'm connected to the internet

So that said, it should be pretty clear how I feel about emailing a document back and forth. Some tools like Google Drive, Dropbox, or Box solve some of the problems with versioning/naming and make it much harder to permanently delete something. But they don't do a great job of avoiding duplication and version reconciliation if multiple authors are working on a document at the same time. I do think these have a place in collaborative research (see below), but they also don't entirely fit the conditions laid out above.

I think it may be useful to think about collaboration in research as involving three types of documents: data, code, and writing.

## Collaborating on Data
This is the area in which a shared drive in the cloud makes the most sense to me. Once it has been collected, your data should not change. You may end up manipulating it a lot for your analysis, but that should all be done in code, not in the original data itself. A cloud storage option will make the file accessible to all collaborators and represents one backup.[^1] Cloud storage is also good for data because many times your data may be stored in a very large file (or multiple large files) and is thus less suitable to other sharing methods.

There are, I think, two downsides to this option. First, if you aren't going to use the one cloud storage option for all types of collaborate files (and I certainly don't want to for my work), then it's just one more tool to try to keep track of. Second, depending on the way you set up your workflow, if these files do ever change, then it may be difficult for you to detect that change.

## Collaborating on Code
The more that I do statistical work, the more convinced I am that the only way to share code is by putting it under version control and using [GitHub](https://github.com) (or another similar service like [GitLab](https://gitlab.com)). These tools were built specifically for collaborative work on computer code, so it shouldn't be a surprise that they work well. The advantages to this approach are enormous and others have already done a great job of laying out the case for using these tools.[^2] For now I'll just say that using git and GitHub perfectly fit the conditions I laid out above for what I look for in collaborative authoring tools.

## Collaborating on Writing
Writing is probably the trickiest option here and will depend largely on the exact nature of the product and the knowledge of your collaborators. For academic writing, I prefer to use LaTeX and because those are just plain text files, they play nicely with git as well. So my strong preference for this aspect is, again, to use version control (with git) and either GitHub or GitLab to share. But having used this on a couple of projects, I can say that it works best when all users have sufficient knowledge of git, and there is a bit of a learning curve with it. The two other methods I've used are ShareLaTeX and Google Docs. Both of them allow authors to work simultaneously in "real time." This is a feature that doesn't really appeal to me for a variety of reasons. My experience has been that ShareLaTeX is slow (which shouldn't be too surprising since it's constantly trying to recompile TeX). Both ShareLaTeX and Google Docs run through your browser, anything you do there will work for all of your collaborators, regardless of their systems. But that aspect also violates my last *don't* from above about requiring an internet connection.

# Closing Thoughts
These tools and this setup has worked well for me, but I recognize that it won't work equally well for everybody. But I think these final suggestions should generalize:

  1. Use as few different systems of tools as necessary to accomplish your task. If you can do everything using only Box and you and your collaborators are all satisfied with the workflow and the results, then only use Box. The more systems you involve, the more opportunities there are to create confusion. But if your systems aren't doing what you need them to do, find the tools that will and add those in. (I suppose this is the Occam's Razor of computer advice.)
  2. Make sure you have backups of everything. Google is not infallible.
  3. Learn as many of these tools and systems as early as possible, even if you don't use them immediately or regularly. Having that knowledge will give more flexibility in future collaboration and might save you headaches down the road because in the end, what you really want, is a system of collaborative tools that allow you to work with others seamlessly.

[^1]: Note that I say "one" backup and not "the" backup. You need [more than one](http://www.hanselman.com/blog/TheComputerBackupRuleOfThree.aspx) backup. Period.
[^2]: See, for example, Alex Branham's [series of posts](http://www.jabranham.com) on the subject.
