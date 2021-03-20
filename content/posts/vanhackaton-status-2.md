---
title: 'Vanhackaton status 2'
date: Sun, 23 Oct 2016 00:38:18 +0000
draft: false
tags: ['express', 'inline-edit', 'nodejs', 'sqllite', 'status', 'thinkific', 'vanhackaton']
---

![2016-10-22_2225](/images/2016/10/2016-10-22_2225.png) Today was a very productive day!! tons of nice features add to the editor:

*   Converted all the javascript code to run inside an Angular application
*   Create the UI to edit the questions, with add and remove
*   Created the timeline that shows each question position in the video
*   Created backend using node.js and SQLLite
*   Add inline edit to the story title and questions
*   Scroll to the question when clicks on the timeline marker
*   Monitor the execution of the video and pause for the question, tracking the answer in the edit mode

Getting close to the first usable version! next steps:

*   Play the story in separated page
*   "Login" on the first page by entering the email
*   Replace the save button by calls to save on any changes made to the story
*   Deploy to a server