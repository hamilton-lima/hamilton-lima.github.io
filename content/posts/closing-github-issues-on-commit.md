---
title: 'Closing github issues on commit'
date: Sat, 29 Oct 2016 19:13:14 +0000
draft: false
tags: ['automation', 'commit', 'git', 'github', 'Uncategorized', 'video-editor']
---

There is no better place to keep track of opensource projects issues, than the issues page at github :) like this one: [https://github.com/hamilton-lima/vanhackaton-video-editor/issues](https://github.com/hamilton-lima/vanhackaton-video-editor/issues) And as expected there is an automated way to close the issues with comments from your commit messages. You can use several key words for that as github documentation explains here: [https://help.github.com/articles/closing-issues-via-commit-messages/](https://help.github.com/articles/closing-issues-via-commit-messages/), can be: close, closes, closed, fix, fixes, fixed, resolve, resolves, resolved. To fix #4 & #6 I used the following commit message:

> This closes #4, closes #6

It looks human readable and closes the issues as well, pretty good isn't it ? ![2016-10-29_1705](/images/2016/10/2016-10-29_1705.png)