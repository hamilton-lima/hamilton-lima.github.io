---
title: 'Vanhackton - Lessons Learned'
date: Wed, 26 Oct 2016 23:00:17 +0000
draft: false
tags: ['500', 'after battle', 'angular', 'dragable', 'easejs', 'express', 'javascript', 'lessons', 'nodejs', 'numeraljs', 'sqllite', 'timeline', 'vanhackaton', 'video', 'vote']
---

![National Monument at Kuala Lumpur, Malaysia](/images/2016/10/2014-06-22-18.26.36.png) After a very intense weekend I feel like coming from the battle :) But it feels very good to deliver a good result in such a small timeframe. On this post I will describe some lessons learned from the past weekend and some decisions I made that worth.

POC - the start
---------------

For every project, I allways plan for proof of concepts, small implementations that isolate the implementation complexity by implementing just the necessary, to solve one problem. On this case the problem to solve was:

> How to show a DIV over the youtube video

One mistake that I believe I made was that during the research of existing solutions, I only evaluated the UI itself, didn´t look at the page source code to check how they did it. That result in some extra hours to understand how to do it and to implement it. End of the day it worked fine, but it would be better if I read that source codes :) See [index.css](https://github.com/hamilton-lima/vanhackaton-video-editor/blob/master/server/client/css/index.css) at #video and #video #questions about the solution used to show the div over the video.

The timeline control
--------------------

![2016-10-26_2011](/images/2016/10/2016-10-26_2011.png) The timeline control was a tricky part of the implementation, first I created a div for the white background, then small blue divs over it at the proportional position to show the question timeline. Everything was going until the moment that I felt like I need to drag those divs to change the question second. After trying a lot to use [dragable()](http://api.jqueryui.com/draggable/#option-containment) I wasn´t able to make it work inside the div with "position: absolute". So I changed plans and created a [EaseJS](http://www.createjs.com/easeljs) [stage](http://www.createjs.com/docs/easeljs/classes/Stage.html) to draw the timeline and allow the user to drag it and change the question second by dragging it. I know that I should have a little message pushing the user to drag it, or at least change the cursor to indicate that the timeline bars are dragable, but this I realize after everything is finished :) To see the timeline implementation, look for it at [app.js](https://github.com/hamilton-lima/vanhackaton-video-editor/blob/master/server/client/app/app.js) at the directive "draggableTimeline", that of course should be in a separated file.

The database choice
-------------------

My first option was to save all the data in flat files, using folders to separate the stories for each user, and to keep one master pointing for the files in the folders, but I drop this idea really quick, when I start to remember how flat files handling can be a pain, specially about zombie processes locking files. So instead of going in the direction of setting an [NoSQL DB](http://nosql-database.org/), or an relational like [Mysql](https://www.mysql.com/) or [Postgres](https://www.postgresql.org/), I decided to keep it as simple as possible by going with [SQLLite](https://sqlite.org/). That decision saved me some hours of handling issues like: server setup, user creation, access rights. Everything was plain simple. It was just a matter of write a code to create the table, and some code to save data, yes plain INSERTs, and read data when need, using plain SQL. One main improvement that I can see from the implementation is to separate the database access details in a separated component, So it would be easier to plug other database imeplemention without changing everything. One detail that I believe it worth highlight is that the query function are asynchronous, I gave a try on some implementations using [Promisses over SQLLite](https://github.com/kriasoft/node-sqlite), but I didn't like the syntax at all, It looks confusing to me the end result, this could be result of my stress during the weekend, or quick judgement to avoid potential risks. End of the day [implemented a simple solution to handle it.](https://github.com/hamilton-lima/vanhackaton-video-editor/blob/master/server/index.js)

```
app.get('/api/story/:id', function (req, res) {
	var msg = 'get story from id: ' + req.params.id;
	console.log(msg);

	var result = db.all('SELECT \* FROM story WHERE id = ? ', req.params.id, 
	**function(err,rows){**
		if(err){
			console.log('error: ', err );
  			**res.status(500).send({ error: err });**
  			return;
  		}

		console.log(JSON.stringify(rows));
		res.json(rows);
	});
});
```

In this fragment of code there is a pitfall, that got me several times, that is to forget the return; just after the .status(500).send(...), I know that the send() method does not end the the function, but somewhere in my brain that line is wired as final, because we literally cannot send anything else as result after that.

Components Hall of fame
-----------------------

These are the shiny little guys that made my life easier!

### Client side

- Youtube player API [https://developers.google.com/youtube/iframe\_api\_reference#Playback\_controls](https://developers.google.com/youtube/iframe_api_reference#Playback_controls) 
- Angular youtube component [https://github.com/brandly/angular-youtube-embed](https://github.com/brandly/angular-youtube-embed) 
- Number format (seconds to hour:minute) just lovely <3 [http://numeraljs.com/](http://numeraljs.com/) 
- Angular inline edit [https://github.com/tameraydin/ng-inline-edit](https://github.com/tameraydin/ng-inline-edit) 
- Angular scroll [https://github.com/oblador/angular-scroll/](https://github.com/oblador/angular-scroll/) 
- Angular UI bootstrap [https://angular-ui.github.io/bootstrap/](https://angular-ui.github.io/bootstrap/)

### Server side

- Install express routing [https://expressjs.com/en/starter/installing.html](https://expressjs.com/en/starter/installing.html) 
- Install sqllite3 [https://www.npmjs.com/package/sqlite3](https://www.npmjs.com/package/sqlite3) Examples using sqllite3 [http://blog.modulus.io/nodejs-and-sqlite](http://blog.modulus.io/nodejs-and-sqlite)
- Install body parser (don´t forget this one) [https://github.com/expressjs/body-parser](https://github.com/expressjs/body-parser) 
- Sqllite navigator [http://sqlitebrowser.org/](http://sqlitebrowser.org/) Icons Used as favicon one icon made by [Madebyoliver](http://www.flaticon.com/authors/madebyoliver) from [http://www.flaticon.com](http://www.flaticon.com) Licensed by [http://creativecommons.org/licenses/by/3.0](http://creativecommons.org/licenses/by/3.0)/

Project Links
-------------

- Project pitch: [https://youtu.be/Wx09vx0tZd0](https://youtu.be/Wx09vx0tZd0) 
- Live demo: [http://hamiltonlima.com:3000](http://hamiltonlima.com:3000) 
- Source code: [https://github.com/hamilton-lima/vanhackaton-video-editor](https://github.com/hamilton-lima/vanhackaton-video-editor) 
- Project pitch in portuguese: [https://www.youtube.com/watch?v=vcZZ0uiL2Ns](https://www.youtube.com/watch?v=vcZZ0uiL2Ns)

Vote
----

If you like the project, goto to the url bellow and click on WANT to vote for it. [https://wantoo.io/vanhackathon/5561/video-editor/](https://wantoo.io/vanhackathon/5561/video-editor/) [![vote](/images/2016/10/vote.jpg)](https://wantoo.io/vanhackathon/5561/video-editor/)