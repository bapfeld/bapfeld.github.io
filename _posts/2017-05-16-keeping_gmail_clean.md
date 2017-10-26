---
layout: post
title: "Keeping Gmail Clean"
categories: [Workflow]
---
This post isn't exactly related to R or political science, but it's part of my workflow, so I thought I'd share. If you're like me, then having unread messages is bad. As that little number in my inbox grows, so does my stress. And it's not just the inbox that I think should be free of unread messages, it's *all* folders. Including the "all mail" folder where Gmail sends you messages to live forever. Some of you may think I'm insane and that's fine, you can stop reading now. But if you're like me, then this may make your life just a little nicer.

The problem, I find, is that there are messages that I want to archive without ever opening. I use Mail on the Mac (which is hugely flawed, I know), so when a new message comes in, I get a little banner notification that has the option to delete (re: archive) the message instantly. I use this feature a lot and it's fantastic. The subject line and first line tell me I don't need to read the email, I hit delete, and it's gone. The only problem is that it's left as unread. If you use any mail client, including something like the Mail app on iOS, then you could find yourself in a similar situation.

Enter the solution: a Google script that marks any archived unread messages as read. To set it up, follow the directions that [Mike Crittenden](http://mikecr.it/ramblings/marking-gmail-read-with-apps-script) put together. Here's the actual script:

```javascript
function markArchivedAsRead() {
  var threads = GmailApp.search('label:unread -label:inbox');
  GmailApp.markThreadsRead(threads);
};
```

It's just this: you define a function called whatever you want (in this case "markArchivedAsRead"). You create a variable called "threads" that's the result of searching your gmail for any messages that has the label "unread" and is NOT in the inbox (that's what the "-" is for in front of `label:inbox`). Then the last line marks as read any of those messages.

Okay, so again, follow those directions on what to actually do with this code and where to put it. I set mine to run once a day in the middle of the night, but you can certainly configure it to run more frequently.

Two really important notes that I've learned the hard way:
1. If you have *any* rules setup for your account that involve skipping the inbox, you'll need to change the `GmailApp.search()` call or else those will also be marked as read. For example, when I was TAing for the huge online class, I created a rule for emails that came in from our proctor service. The filter skipped the inbox and added the label "ProctorU", so I had to add `-label:ProctorU` to my code. I once ignored an email for nearly a year because I didn't realize that I'd created this trap for myself. Don't be that person.
2.(Updated) If you ever change your password, you'll have to go back in to the script and reauthorize it. This is relatively straightforward. You should receive an email the first time the script fails (and every time after that), so if you forget about this, Gmail will remind you. Open the script, and run it. You'll be prompted to authorize and you should be good to go after that. You will also need to clear the old trigger and set a new one.
