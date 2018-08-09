---
layout:		post
title:		"DraftMEAN"
image:		draftmean
tags:		[web]
priority:   1

---
# Fantasy Football Draft Board

After taking a bit of time away from web development to focus on my full-time job, I decided it was time to come up with an idea for an app. I had a few requirements:

1. Must be something that I can work on in my spare time, independent of any client's needs or timelines
2. Must provide value to someone, even if it's just myself
3. Must require that I learn something new, preferably a new language or tech stack
4. Must require more time spent on back-end logic than the actual web design

Last year in 2017, the idea hit me right around August during the start of fantasy football season. My dad has run our family + friends fantasy football league for over a decade now, and every year he's insisted on using a giant poster board for managing our league's draft every year, even in the face of ESPN and Yahoo websites that continue to build better and nicer online pages. One of the main reasons<sub><sup>**</sup></sub> is that he appreciates the authenticity and locality of it; everyone has to gather together in a common location, bring their own draft materials and rankings (even though most people don't and he usually has to print them out anyway), take frequent breaks to eat pizza, and draft their players by shouting names and positions to whoever doesn't have a team but is still fortunate enough to be allowed to participate in the process. Switching to ESPN's managed draft would remove all of that authenticity and enable anyone to just draft from the comfort of their home, which isn't fun at all. However, this past year people started making comments about how kids these days could probably figure out a way to digitize the whole process, and "could probably even project it up on the wall."

<sub><sup><sub><sup>**Disclaimer: There are actual legitimate reasons as well, mainly that most of the 16ish people in our league are kids and technology-challenged people, and getting everyone to bring laptops or phones and log into their ESPN app is a bit of a challenge. The "authenticity" reason is a lot more fun though.</sup></sub></sup></sub>

It hit me that I could probably build something that more than qualifies as a fantasy football draft board, and since part of the reason that more popular draft boards weren't being used was because of their complexity and barriers to entry, lack of complex features like login accounts and league management would actually be a feature of the tool. The more I thought about it, the more it made sense; no one was asking for this specifically, so I didn't have any specific timelines (req #1), but it would definitely make things easier on everyone (req #2). Additionally, one of the features that would be critical to making it usable is web sockets, and the new MEAN stack (MongoDB, ExpressJS, AngularJS, NodeJS) is perfectly suited for supporting web sockets (req #3). Last, if I'm building this entire draft board framework in the backend on a MEAN stack, it'll require a lot of real coding before I can touch any of the design (req #4). It was perfect.

Over the past several months, I've actually built a working application that I'm really excited about, and I'm finally ready to publish it to the world at [www.draftmean.net][dratmeanhome]. If you'd like to test out the fun part, you can play with [a sample draft board that I've created][draftmeansampleboard]. It may not be an app at the scale of something like ESPN, but at its core is a web app that allows multiple users to view the same draft board, select any given player, draft them to the next upcoming team, and - through the magic of web sockets - watch new draft picks update in real-time on everyone's screens. In the process, I learned Angular CLI (or 4 or IO, I'm still not sure what the common convention to call it is), NodeJS with ExpressJS handling the non-socket API endpoints, socket.io for managing everything having to do with drafting a player, and MongoDB to store all of the players and draft boards. My dad loves it and thinks it's perfect for our league, and is excited to use it this coming year. To top it all off, I've built my first real application outside of my job that has value and I can be proud of.

# Code

Oh yes, there is most definitely code! 

## Web App

First of all, I actually have two separate projects. This is because Angular 4, unlike Angular 1, is actually a standalone framework installed and managed through NPM, and it typically likes to run on its own port. Obviously you can run Angular 4 apps in the same directory and project as your Node app, but it would have been a very minor headache to configure, which was enough of a headache for me to work around. As a result, the actual web application lives at [DraftMean-Web][draftmeanweb], while the API is managed at [DraftMean.API][draftmeanapi].

As for the code itself, it's a little jacked up because I've been adjusting everything to fit the new Create component for creating new draft boards, but I can still show some of the guts. Basically, each of the projects break down into little sub-sections that handle different parts of the application. For the Angular app, at the top is the AppModule, a router to tie it all together, and then the components. Since I'm building multiple pages with different layouts, the AppComponent is basically just a pass-through:

[app-component.html][appcomponenthtml]
{% highlight html %}
<div class="menu">
    <span class="logo">
        Draft<span>MEAN</span>
    </span>
</div>
<router-outlet></router-outlet>
<!-- <app-create></app-create> -->
{% endhighlight %}

You can see where I've previously been keeping the hardcoded component tags on the app component for simplicity, but now everything just goes where the router says to go, and the sub-components handle all of the display logic from there.

From there, let's say that you get into the actual draft board. The HTML is all pretty optimized. A lot of \*ngFor loops, which is normal. There are also even more sub-components, which I'll get into in a minute.

[board.component.html][boardcomponenthtml]
{% highlight html %}
<div class="loading" [class.finished]="boardLoaded&&playersLoaded">
  <mat-progress-spinner
      [color]="color"
      [mode]="mode"
      [value]="value">
  </mat-progress-spinner>
  <span>
      Getting your draft board ready...
  </span>
</div>
<div class="section draftboard twelve">
  <div class="roundsHeader">
    <div class="object">
      <span>Rounds</span>
    </div>
  </div>
  <div class="row header">
  <div *ngFor="let team of teams" class="object">
    <span (click)="displayTeam(team.id)">{{ team.name }}</span>
  </div>
</div>
  <div class="row first">
    <app-draftpicks *ngFor="let pick of picks"
      [pick]="pick" [players]="draftedPlayers">
    </app-draftpicks>
  </div>
  <div class="rounds">
    <div *ngFor="let round of roundNumbers" class="object">
      <span>{{ round }}</span>
    </div>
  </div>
</div>
<div class="section players">
  <div class="container">
      <div class="header">
          <h1>Available Players</h1>
      </div>
      <app-players [allPlayers]="playersList" [socket]="socket"></app-players>
  </div>
</div>
{% endhighlight %}

Behind the scenes, the TypeScript class for the BoardComponent is where I feel like most of the fun really happens. You can view all of the code by clicking on the file name below, but I've pasted some of the best snippets below. My two favorite parts are 1) this.playersLoaded = true once the players are loaded from the API, which, if you notice in the HTML above, means the loading screen will display right up until the API sends its response asynchronously, when it fades away; and 2) the displayTeam function, where I came up with this really cool formula to figure out who had been picked by any given team in the board, which is important because, as you'll see and have explained later, the data model doesn't include any direct links between a team and all their picks.

[board.component.ts][boardcomponentts]
{% highlight js %}
ngOnInit(): void {
    const id = this.route.snapshot.paramMap.get('id');

    // Initialize board, teams, and vars for board settings
    this.boardService.getBoard(id).subscribe(
      board => {
        this.board = board;
        this.teams = this.board.teams;
        
        this.totalRounds = this.board.totalRounds;
        this.totalPicks = this.teams.length * this.totalRounds;
        this.picks = Array(this.totalPicks).fill(1).map((x,i)=>i+1);
        this.roundNumbers = Array(this.totalRounds).fill(1).map((x,i)=>i+1);
        console.log('board loaded');
        this.boardLoaded = true;
      }
    );

    // Pull in all the players
    this.playerService.getPlayers(id).subscribe(
      players => {
        this.playersList = players;
        console.log('players loaded');
        this.playersLoaded = true;
      }
    );
    this.socket = io(this.playerService.apiUrl);
  }

  ngDoCheck() {
    if (this.playersList) {
      // Socket stuff
      let _this = this;
      this.socket.on('PlayerUpdated', function(data) {
        console.log('PlayerUpdated: ' + JSON.stringify(data));
        var newPlayer = data.updatedPlayer;
        var playerToUpdate = _this.playersList.find(player => player.Rank == newPlayer.Rank);
        var updateIndex = _this.playersList.indexOf(playerToUpdate);
        console.log('updateIndex: ' + updateIndex);
        _this.playersList[updateIndex] = newPlayer;
      });

      // Set draftedPlayers for DraftPicks component
      this.draftedPlayers = this.playersList.filter(player => player.PickTaken > 0);
    }
  }

  displayTeam(pickNumber = 12) {
    this.teamPlayers = [];
    var playerPicks = Array(this.totalRounds).fill(0).map(
      (x,i) => i * this.teams.length + pickNumber
    );

    playerPicks.forEach(function(e, i, arr) {
      /*
       * Outcome: Snake pick
       * Logic:
       *  e:                element
       *  i:                index
       *  if statement:     only change every other element
       *  1st parentheses:  get last pick of this round (e.g. 36)
       *  2nd parentheses:  get picks per round by dividing last pick by index, e.g. 36 / 3 = 12
       *  overall:          last pick (36) plus ppr (12) = 48 less pick number (4) = 44 plus one = 45
       */
      if (i % 2 != 0) { arr[i] = ((e-pickNumber)+((e-pickNumber)/i)-pickNumber+1) };
    });

    playerPicks.forEach(element => {
      var player: Player = this.playersList.find(player => player.PickTaken == element);
      if (player) this.teamPlayers.push(player);
    });

    this.dialog.open(TeamsDialog, {
      data: this.teamPlayers
    });
  }
{% endhighlight %}

While I don't have space to get into the code for every component, there are more important pieces. The service ([player.service.ts][playerservicets]) handles all of the asynchronous gathering of data through Observables and socket-y updating of data through socket.io's Angular plugin. As I pointed out above, the board component has two important sub-components: one for [displaying all of the availble players left in the pool][playerscomponenthtml], and one for [displaying each individual player][draftpickcomponenthtml], which might be overkill but I didn't realize that you could create component templates until a few weeks ago and don't feel like rewriting this just yet. Lastly there are all of the Angular models, e.g. [player.ts][playerts].

You can see that one of my main goals was to try to keep most of the business logic on the front-end, rather than create a million API endpoints for all of the different variations of returning a list of players and filters to put on them. I'm not sure if that makes me a bad microservices developer, a normal and sensible developer, or both, but I think it makes sense.

## API

That leaves us with the API, and this is mostly straightforward. Like in Angular apps, Express APIs are generally broken into sub-sections that you can build out horizontally. The smallest layer is the model, which is built using Mongoose to handle most of the actual object modeling. Note: this is basically the only document model that I use in the entire application.

[player.model.js][playermodeljs]
{% highlight js %}
var mongoose = require('mongoose');
var mongoosePaginate = require('mongoose-paginate');

var PlayerSchema = new mongoose.Schema({
    Rank: Number,
    PlayerName: String,
    Team: String,
    Position: String,
    ByeWeek: Number,
    BestRank: Number,
    WorstRank: Number,
    AvgRank: Number,
    StdDev: Number,
    ADP: Number,
    // IsDrafted: Boolean,
    PickTaken: Number,
    BoardId: String
}, {
    collection: 'players'
});

PlayerSchema.set('toJSON', {
    virtuals: true
});

PlayerSchema.plugin(mongoosePaginate);
const Player = mongoose.model('Player', PlayerSchema);

module.exports = Player;
{% endhighlight %}

Out of the 12 fields in the document model, ten of them are essentially just stats about the player. The only two fields that I'm using to tie players to the whole draft board system, including which board is using them and which team picked which player(s), are BoardId and PickTaken. Just with this information I'm able to build out the entire Angular application above. The reason I've done this instead of build out some highly normalized relational database is because MongoDB - and NoSQL DBs like it - are meant to be document-based and rely on relationships as little as possible. However, documents are notoriously difficult to frequently edit and update without performance concerns, so instead of build a single document for each board and then update that board document every single time a player is drafted or updated, I can just populate an entire list of all the players that could be taken, and then only update each player once they've been drafted on whatever board they're assigned. If you ask me, it's pretty freaking clever.

Anyway, we need to define some actions for our model, and wrap them in useful endpoints, and this is where the second level comes into picture: the service. Essentially, we're just safety-checking objects before they get sent to Mongoose's methods, which in this case are paginate and save.

[player.service.js][playerservicejs]
{% highlight js %}
var Player = require('../models/player.model');

_this = this;

exports.getPlayers = async function(query, page, limit) {
    var options = {
        page,
        limit
    };

    try {
        var players = await Player.paginate(query, {
            page: page,
            limit: limit,
            sort: { "Rank": 1 }
        });
        return players;
    } catch (e) {
        throw Error('Error while paginating Players');
    }
};

exports.updatePlayer = async function(player) {
    var rank = player.Rank;
    var boardId = player.BoardId;
    console.log(player);

    try {
        var oldPlayer = await Player.findOne({ "Rank": rank.toString(), "BoardId": boardId });
    } catch (e) {
        throw Error("Error occurred while finding the player");
    }

    console.log(oldPlayer);
    if (!oldPlayer)
        return false;
        
    console.log(player.PickTaken);

    oldPlayer.PickTaken = player.PickTaken;

    console.log(oldPlayer);

    try {
        var savedPlayer = await oldPlayer.save();
        return savedPlayer;
    } catch (e) {
        throw Error("Error occurred while updating the player: " + e.message);
    }
}
{% endhighlight %}

The next level up is the controller, and this is essentially where we hook Express into our model, although it's not explicitly stated that way in the script itself. You might notice that this is starting to look like a really weird MVC framework, except the V is more of an S for Service, and you'd kind of be right. Also, this is where you'll finally start seeing the first signs of some endpoints using Express and others using socket.io.

[player.controller.js][playercontrollerjs]
{% highlight js %}
var PlayerService = require('../services/player.service');

_this = this;

exports.getPlayers = async function(req, res, next) {
    var page = req.query.page ? req.query.page : 1;
    var limit = req.query.limit ? Number(req.query.limit) : 30;

    var boardId = req.params.boardId;

    if (boardId) {
        try {
            var players = await PlayerService.getPlayers({ "BoardId": boardId }, page, limit);
            return res.status(200).json({
                status: 200,
                data: players,
                message: "Successfully received players"
            });
        } catch (e) {
            return res.status(400).json({
                status: 400,
                message: e.message
            });
        }
    } else {
        return res.status(400).json({
            status: 400,
            message: 'Please provide a Board ID in the request parameter.'
        });
    }
}

exports.updatePlayer = async function(io, Player) {
    var result;
    
    if (!Player.Rank) {
        result = {
            'success': false,
            'message': 'Update failed',
            'error': 'ID/Rank must be present'
        };
        io.emit('PlayerUpdated', result);
    }

    var id = Player.Rank;
    console.log(Player);

    try {
        var updatedPlayer = await PlayerService.updatePlayer(Player);
        result = {
            'success': true,
            'message': 'Successfully updated player',
            'updatedPlayer': updatedPlayer
        };
        io.emit('PlayerUpdated', result);
    } catch (e) {
        result = {
            'success': false,
            'message': 'updatePlayer failed',
            'error': e
        };
        io.emit('PlayerUpdated', result);
    }
}
{% endhighlight %}

Last is the route that actually attaches an HTTP endpoint to our Express methods. You can see that I've commented out the 'put' method to avoid anyone using standard HTTP requests to update players, and you'll see what I've replaced that with next.

[player.route.js][playerroutejs]
{% highlight js %}
var express = require('express');
var router = express.Router();

var PlayerController = require('../../controllers/player.controller');

router.get('/:boardId', PlayerController.getPlayers);
// router.put('/', PlayerController.updatePlayer);

module.exports = router;
{% endhighlight %}

That essentially covers the core of a Node + Express (+ Mongoose) API, but it's time to finally fit the web socket framework in. This was one of the parts that took me a while to figure out - not because socket.io is very difficult to use, but because the way it's meant to be used doesn't fit very well with most of the auto-generated code that NPM gets you for a Node + Express app. The general idea is to tie everything in - your MongoDB connection, all of your imports, the initialization of your Express app - in the app.js module, and then dump the Express app as one giant module for /bin/www to import and tie to a port. However, socket.io needs you to tell it which HTTP server to bind itself to, which only happens in /bin/www after your Express app has been bound to a new HTTP server. My trick to make all of this clean was to build a new module specifically for socket.io, but have the initialization of the module require an HTTP server as its dependency. Then, in /bin/www, simply initialize my socket module by passing in the newly-created HTTP server, and move on the same as before.

If you've ever had one of those moments where you come up with a really good idea but expect it to fail massively because it seems too simple, this was my moment, but it actually worked flawlessly.

[socket-server.js][socketserverjs]
{% highlight js %}
module.exports = function(server) {
    var module = {}

    var io = require('socket.io')(server);
    var PlayerController = require('./controllers/player.controller');

    io.on('connection', function(socket) {
        socket.on('updatePlayer', function(Player) {
            console.log('socketData: ' + JSON.stringify(Player));
            PlayerController.updatePlayer(io, Player);
        });
    });

    module.io = io;

    return module;
}
{% endhighlight %}

[/bin/www][www]
{% highlight js %}
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var app = require('../app');
var debug = require('debug')('draftmeanapi:server');
var http = require('http');

/**
 * Get port from environment and store in Express.
 */

var port = normalizePort(process.env.PORT || '3000');
app.set('port', port);

/**
 * Create HTTP server.
 */

var server = http.createServer(app);

/**
 * Import Socket.IO with server dependency
 */

 var io = require('../socket-server')(server);

/**
 * Listen on provided port, on all network interfaces.
 */

server.listen(port);
{% endhighlight %}

One final thing that is interesting to point out is that the actual "socket server" is only listening to one request - the updatePlayer message - and then passing the responsibility off to the controller I defined earlier. If you look back at the update method in that controller, you can see that at the very end it's emitting a new message with the updated record for whoever is on the other side to listen to. This is essentially the core of how the API and all open web pages interact with each other, and it works like a charm!

[draftmeanweb]:             https://github.com/lilweirdward/DraftMean-Web
[draftmeanapi]:             https://github.com/lilweirdward/DraftMean.API
[draftmeanhome]:            https://www.draftmean.net
[draftmeansampleboard]:     https://www.draftmean.net/board/W2vAny56RwAgYTs1
[appcomponenthtml]:         https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/app.component.html
[boardcomponenthtml]:       https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/board/board.component.html
[boardcomponentts]:         https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/board/board.component.ts
[playerservicets]:          https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/player.service.ts
[playerscomponenthtml]:     https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/board/players/players.component.html
[draftpickcomponenthtml]:   https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/board/draftpicks/draftpicks.component.html
[playerts]:                 https://github.com/lilweirdward/DraftMean-Web/blob/master/src/app/models/player.ts
[playermodeljs]:            https://github.com/lilweirdward/DraftMean.API/blob/master/models/player.model.js
[playerservicejs]:          https://github.com/lilweirdward/DraftMean.API/blob/master/services/player.service.js
[playercontrollerjs]:       https://github.com/lilweirdward/DraftMean.API/blob/master/controllers/player.controller.js
[playerroutejs]:            https://github.com/lilweirdward/DraftMean.API/blob/master/routes/api/players.route.js
[socketserverjs]:           https://github.com/lilweirdward/DraftMean.API/blob/master/socket-server.js
[www]:                      https://github.com/lilweirdward/DraftMean.API/blob/master/bin/www