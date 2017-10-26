---
layout: post
title: "LaTeX and PDFs"
tags: [workflow]
---

I've used LaTeX for writing nearly everything since starting in grad school. There are a huge number of advantages of doing this, but there are also plenty of frustrations. One of these problems is compiling the document. Any time that I want to see an updated file, I have to manually compile the thing into a PDF. With internal references and citations, this often means compiling the document multiple times. This is a waste of time and a real pain. Enter `latexmk`. The solution I lay out here works really well for me because I'm not interested in using emacs. I know the same result is possible in that setup, but I don't want to go down that rabbit hole.

The `latexmk` package, which ships with full distributions of TeX simplifies everything in one step- it compiles your document as many times as necessary to resolve all references, citations, etc. Following the helpful advice [here](http://jon.smajda.com/2008/03/08/latexmk/) this workflow also allows *live* updating, another advantage over the ordinary LaTeX setup. In addition to saving time and a headache, this can be a really useful tool for tweaking graphics and building figures, for example with tikz.

First, I open the TeX file in a text editor. I'm using Atom (which has other advantages I'll describe in a second), but you can use any one that you like. Next, I open the PDF in a viewer. Following Jon Smajda's advice, I'm using [Skim](http://skim-app.sourceforge.net) because it can update the open file without having to be the active application (in contrast to Mac's built in Preview.app).  

Now for the fun part. I open a Terminal window and run the command `latexmk -pdf -pvc <filename.tex>`. This will build the pdf and then stay running in the background. Anytime it detects changes to the TeX file, it will update the preview. I can then switch into a full screen mode where the text editor sits on one side and the preview sits in another. Every time I save my changes, the file updates without any further input from me.

There's some more advanced stuff you can do as well to simply this process further. First, I created a script to automate opening the files and getting latexmk running in the background. The script is titled "WritingProject.sh" and looks like this:

```
cd ~/Documents/Location/Of/My/Project/;
open -a Atom.app Project_Tex_File.tex;
open -a Skim.app Project_PDF.pdf;
latexmk -pdf -pvc Draft_Proposal.tex;
```

Whenever I want to get started on my project I just run `sh WritingProjct.sh` from the terminal. At the time of writing I'm only using this setup for one project. Once I start to use this for multiple writing projects at the same time, I think that I'm going to add one more step. I'll create an alias in the `.bash_profile` that reads `alias writing='cd ~/writing_scripts/; ls` so that I can just type `writing` when I start and then choose the script I want from there.

If you use Atom and Skim as I do in this setup, you can simplify this whole process. In Atom you can install the `latex` package and simply use "control + option + b" to (save and) build your file. Skim will then automatically update the preview. You can also use "control + option + c" to clean up all of the extra files that compiling LaTeX documents creates. Unfortunately, the default is to also erase the pdf that you just made. To change this behavior, go to packages in the settings and select latex. Then open the code with the "view code" button. Open the package.json at the root level and within "cleanExtensions" remove .pdf and add .dvi.  Problem solved.
