---
layout: post
title: "Avoiding Headaches when Lecturing with Videos"
categories: [Teaching]
tags: [TAing, Code]
---
We've all sat in classrooms where the instructor wants to play a video but struggles with (often basic) technology. It derails the class, makes the instructor look bad, and is painful to sit through. The result is the same regardless of where the blame lies. And sometimes it really isn't the instructor's fault --- I doubt Murphy's Law is more true in any area than it is in teaching.[^1] So with that scenario in mind, here's a quick post on one measure instructors can take to avoid problems during class.

My rule: if a video is going to be used in a lecture, do not stream it. Too many things can go wrong with streaming. My solution: download the videos ahead of time.

So how do you download videos? Enter the wonderfully simple `youtube-dl` tool. Despite it's name it handles videos from just about any online streaming source imaginable. In fact, I have yet to find a site that it doesn't work with.[^2]

# Getting youtube-dl
First, you'll need to install [Homebrew](https://brew.sh) if you don't already have it. Open the Terminal application and type/paste `/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`. When that's done, install `youtube-dl` by typing `brew install youtube-dl`. Voila!

# Using youtube-dl
Now for the fun part: downloading your videos. First, decide where you want to save your videos. Let's assume that it's in your Movies folder. In the terminal, change the working directory to that location by typing `cd Movies`. To download your video there, type `youtube-dl <video_url>`, replacing "<video_url>" with the actual url for the video you want to download. Depending on how you opened the video (following a link from somewhere else vs. search, for example) may end up putting some unnecessary bits into the address. I usually clean this up, although I don't think that's strictly necessary. For youtube, the address will be something like "watch?v=random_string_of_letters_and_numbers". Then there may be "&list=more_letters". You can get rid of any of those & strings (along with the & itself).

`youtube-dl` includes a ton of additional features. For example, it supports downloading all of the videos from a playlist or just extracting audio from the files. You can read up on everything it can do by typing `man youtube-dl` in the terminal.[^3] Be warned, though, that most of the options are pretty technical.

# How/Why I Use This
Okay, so above I suggested that this is a good way to avoid an embarassing technology situation in the classroom. This is true; think about all the things that might go wrong with streaming videos: the internet might be really slow suddenly, the internet might die completely, the video might be deleted, etc. But there are other advantages to this as well. First, you can edit the videos. Maybe you want the first 30 second and the last two minutes. Maybe you need to take out 30 seconds in the middle where John Oliver goes on a profane rant that is at best tangential to the point of the video for some cheap laughs. Whatever it is, doing that ahead of time means a smoother classroom experience and puts you in control. Second, and related to the topic of control, downloading your videos means that you will have an ad-free experience. Regardless of how you feel about advertising in classrooms or issues related to ad-based revenue for content creators, the last thing you want is for an ad for erectile dysfunction medicine to interrupt your class. Because it doesn't matter if you're teaching 9-year-olds or college seniors, it will be an interruption.

So here's my workflow:

1. Find videos to use
2. Download them with this tool
3. Edit as necessary (usually just with Quicktime)
4. Create a video playlist in VLC
5. Before lecture starts, open video playlist and make first video full screen
6. Play videos as needed in lecture

I prefer to use VLC for these videos for a couple of reasons. First, the player can handle just about any type of video files you throw at it. Second, it supports the playlist feature. This makes it easy to work with and hides the number of videos used from public view (that's not something I want to signal to students ahead of time because of its distraction potential). Third, the player has some great settings for how to handle videos in a playlist. I can, for example, set it so that at the end of one video it will advance to the next video, display a blank screen, and NOT autoplay what's next. That means that I don't have to hover over a video trying to hit pause or fiddle around trying to get the next one to play later on.

Is this more work? Maybe, but only marginally. Finding the videos is what takes the most time. Downloading them can take place in the background and making a new video playlist in VLC takes all of thirty seconds. If there are any learning costs involved they are more than paid off by smooth experiences in lecture.

[^1]: In fact, there's probably an argument to be made that in teaching Murphy's law actually deserves a corollary that even things that *cannot* go wrong, will *still* go wrong.
[^2]: Two quick notes: First, this tutorial is intended for Mac users only. I suspect that it works on Linux, but I can't test that at the moment. I have no idea if an equivalent exists for Windows and if it does how to use it. Sorry. Second, I'm not advocating skirting (or abusing or blatantly ignoring) copyright laws here. I'm going to assume that anybody reading this has already given thought to fair use issues and how they relate to any videos they plan to use in their classroom.
[^3]: If you're unfamiliar with manuals in the terminal, the one thing you have to know is to exit you type `q`.
