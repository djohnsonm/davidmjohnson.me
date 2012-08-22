---
layout: post
title: "Roll your own Twitter Feed with Backbone.js"
date: 2012-05-12 09:51
comments: true
categories: [twitter-bootstrap, html5, jQuery, node.js, subtle-patterns, css3]
---

{%img https://dl.dropbox.com/u/10021156/blog/Capture.png %}

Backbone.js is a JavaScript framework that manages complexity on the client while enabling the creation of rich HTML5 applications in a MVC-style fashion. For the more tech-savvy:

{%blockquote%}
Backbone.js gives structure to web applications by providing models with key-value binding and custom events, collections with a rich API of enumerable functions, views with declarative event handling, and connects it all to your existing API over a RESTful JSON interface. Created by THE Jeremy Ashkenas of Document Cloud (creator of CoffeeScript – more on that later).
{%endblockquote%}

##Setting the stage
In this post we’re going to make our own sample Backbone.js app that utilizes Twitter’s search API to scrape tweets and display them with the underscore templating engine and twitter bootstrap themes with a subtle patterns background. Node.js will deliver our static html file while Dropbox will host our JavaScript files (providing us with a make-shift CDN “Content Delivery Network”). Our app will be hosted on www.cloudfoundry.com with www.c9.io as our staging server while using git as our source control.

##Final Result
Usually with tutorials I scroll to the bottom to see if the example is even worth making, then I decide to read the post if it is. So let me save you the trouble of scrolling and just show the final result here. <iframe src="http://ghbtns.com/github-btn.html?user=djohnsonm&repo=Twitter-Feed&type=fork&count=true"
  allowtransparency="true" frameborder="0" scrolling="0" width="95px" height="20px"></iframe>

{%img https://dl.dropbox.com/u/10021156/blog/ff.png %}

##Setting up our server
For those of you that don’t know node.js like me. It’s worth checking out and is compatible on all major operating systems. According to node’s site.

{% blockquote %}
Node.js is a platform built on Chrome’s JavaScript runtime for easily building fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model that makes it lightweight and efficient, perfect for data-intensive real-time applications that run across distributed devices.
{% endblockquote %}

Some users of node are Microsoft, LinkedIn, eBay and Voxer.

For more info on getting started with node, express and npm (node package manager) check out this tutorial. For more info on what exactly node is, check out this link. There are also plenty of other great ebooks on Amazon.

If you’re coming from the Microsoft paradigm think of express as your ASP.NET MVC 3 web app framework. For our example we are only distributing one html file (since our app is a simple single-page app) that needs no server interaction. The code should be very straightforward.

{% codeblock lang:javascript Node.js app%}
// Configuration

app.configure(function(){

  // disable layout
  app.set("view options", {layout: false});

  // make a custom html template
  app.register('.html', {
    compile: function(str, options){
      return function(locals){
        return str;
      };
    }
  });
});

app.configure('development', function(){
  app.use(express.errorHandler({ dumpExceptions: true, showStack: true }));
});

app.configure('production', function(){
  app.use(express.errorHandler());
});

app.get('/', function(req, res){
  res.render("index.html");
});

app.listen(3000, function(){
  console.log("Express server listening on port %d in %s mode", app.address().port, app.settings.env);
});

{% endcodeblock %}

##The Html

{%img https://dl.dropbox.com/u/10021156/blog/hbp-1.png %}

Main highlights here are as follows:

HTML5 boilerplate goodness
Underscore.js Templating
Using a CDN to shorten the load time of your scripts
Easy styling with Twitter Bootstrappin’

{%img https://dl.dropbox.com/u/10021156/blog/bs.png %}

Here we see by appending classes to div’s twitter bootstrap will easily create a persistent header with optional header text.

{% codeblock lang:html Twitter BootStrap Styles %}
<header>
    <div class="navbar navbar-fixed-top">
      <div class="navbar-inner">
        <div class="container-fluid">
          <a class="brand" href="#">Clarity Twitter Feed</a>
          <div class="nav-collapse collapse">
          </div>
        </div>
      </div>
    </div>
</header>
{% endcodeblock %}

{%img https://dl.dropbox.com/u/10021156/blog/u.png %}

Here in our tweet template we iterate through each tweet and display the respective content in tags and append the image url onto the “src” attribute of an tag.

{% codeblock lang:html Underscore.js style application %}
<script type="text/template" id="tweettemplate">

    <!-- nav -->
      <nav>
        <button class="refresh btn btn-success btn-large">
              <i class="icon-white icon-refresh"></i>Refresh
        </button>
        <button class="reverse btn btn-warning btn-large">
              <i class="icon-white icon-resize-vertical"></i>Reverse
        </button>
      </nav>
    <!-- /nav -->

    <!-- List of Tweets -->
        <section id="content">
          <ul id="tweetList">

          <% _.each(tweets, function(tweet) { %>

          <li class="tweet">
            <span>

                <div class="head">
                    <img class="pic" src="<%= tweet.profile_image_url %>" />
                    <p class="userId"><%= "@" + tweet.from_user %></p>
                 </div>

                 <div class="tail">
                    <p><%= tweet.text %></p>
                    <p class="created"><%= tweet.created_at %></p>
                 </div>
                 <hr />
            </span>
           </li>  

          <% }); %>

          </ul>
        </section>
     <!-- /List of Tweets -->
  </script>
 {% endcodeblock %}

##The CSS
{%img https://dl.dropbox.com/u/10021156/blog/sp.png %}

This was a great exercise in learning more about CSS and some of the new CSS3 styles like dropshadow and border-radius. Going forward I’d like to use more style inheritance and eventually use LESS. (Actually, if you convert this CSS to LESS, it will look the exact same.)

{% codeblock lang:css The CSS %}

 body {
  background-image: url('http://dl.dropbox.com/u/10021156/NsLayout/img/white_tiles.png');
  padding-top: 60px;
  padding-bottom: 40px;
  margin: 0 auto;
}

#narrLogo {
  margin-left:34px;
  padding : 10px;
}

#message {
	position:absolute;
	margin-left:152px;
	margin-top:24px;
	color: white;
}

nav {
	width:85%;
	margin: 10px auto;
	padding: 10px;
	background-color: rgba(19,101,53,0.6);
	border-radius:10px;
	margin-bottom:47px;

}

#logo {
	width:50%;
	margin: 0px auto;
}

#tweetList {
	list-style: none;
}

.hero-unit {
	background-color:rgba(0,0,0,0.1);
	display:inline-block;
	padding:20px;

}

.reverse {
	float:right;

}

.container-fluid {
	margin-left: auto;
	margin-right:auto;
	width:600px;
}

.head {
	display:inline-block;
	padding:10px;
	border-radius:5px;
	background-color:rgba(0,0,0,0.1);
	opacity:0.9;

}

i {
	background-position : -435px -119px !important;
}

img.pic {
	box-shadow: 6px 8px 8px #888;
	height:60px;
	width:60px;
}

.tail {
	padding:2px;
}

.userId {
	display: inline-block;
}

.created {
	font-size:14px !important;
	font-weight:bold !important;
	margin-top:-13px;
	background:white;
	display: inline-block;
	border-radius:5px;
	padding: 0px 3px;
	margin-left: 14px;
	opacity:0.9;
}

.userId {
	margin-left:10px;
}
{% endcodeblock %}

##The Model
{% codeblock lang:javascript Backbone.js Model %}
var Tweet = Backbone.Model.extend();
{% endcodeblock %}

Well that was easy. We can define our Tweets field names and default values by passing in a defaults object into the extend() method of the Model class. But since this is JavaScript we can add properties on the fly and default values would not make sense for the properties in this example.

##The View
On initialize the view will call fetch on our collection of tweets which will use the sync method to initiate an ajax call to Twitter’s API.
The resulting data will then be passed through the parse method, into the render method, and applied to the template. It’s best to look at the View while looking at the Collection code. (see below).

{% codeblock lang:javascript Backbone.js View %}
 TweetsView = Backbone.View.extend({
    initialize: function() {        

      _.bindAll(this, 'render');
      // create a collection
      this.collection = new Tweets;
           this.collection.on('reset', this.render);

      // Fetch the collection and call render() method
      var that = this;
      this.collection.fetch({
        success: function (s) {
            console.log("fetched", s);
            that.render();
        }
      });
    },

    el: $('#tweetContainer'),
    // Use an external template

    template: _.template($('#tweettemplate').html()),

    render: function() {
        // Fill the html with the template and the collection
        $(this.el).html(this.template({ tweets: this.collection.toJSON() }));
    },

    events : {
        'click .refresh' : 'refresh',
        'click .reverse' : 'reverse'
    },

    refresh : function() {

     this.collection.fetch();
    console.log('refresh', this.collection);
     this.render();

    },

    reverse : function() {

       this.collection.reverse();
       console.log(this.collection.reverse);
    }

});

var app = new TweetsView();
});

{% endcodeblock %}

##The Collection
Here reverse will override our comparator to display tweets in reverse order by way of the ternary operator provided to us in JavaScript.
{% codeblock lang:js Backbone.js Collection %}
Tweets = Backbone.Collection.extend(
    {
        model: Tweet,
        // Url to request when fetch() is called
        url: 'http://search.twitter.com/search.json?q=claritycon',
        initialize : function(){
            this.sort_order = 'desc';
        },
        parse: function(response) {
 
            //modify dates to be more readable
            $.each(response.results, function(i,val) {
                val.created_at = val.created_at.slice(0, val.created_at.length - 6);
              });
 
            return response.results;
        },
        // Overwrite the sync method to pass over the Same Origin Policy
        sync: function(method, model, options) {
            var that = this;
                var params = _.extend({
                    type: 'GET',
                    dataType: 'jsonp',
                    url: that.url,
                    processData: true
                }, options);
 
            return $.ajax(params);
        },
        comparator: function(activity) {
            console.log("this is comparing");
            var date = new Date(activity.get('created_at'));
            return this.sort_order == 'desc'
                 ? -date.getTime()
                 :  date.getTime()
        },
        reverse: function() {
            console.log("this is reversing");
            this.sort_order = this.sort_order == 'desc' ? 'asc' : 'desc';
            this.sort();
        }
    });

 {% endcodeblock %}

##Refresh and Reverse
When refresh is clicked you can watch the ajax call in Chrome tools.

{%img https://dl.dropbox.com/u/10021156/blog/tweeter.png %}

##A note about Twitter’s APIs

{%img https://dl.dropbox.com/u/10021156/blog/twitter.png %}

The twitter API comes in 1 of 3 flavors. For our example we will use the search API which will allow us to retrieve tweets in jsonp (json with “padding”) and parse them on the client. It’s important to note that we are making a cross-domain request so we have to use jsonp to avoid the “same-origin policy” issue. One thing to also note, the search API only allows for 150 requests per hour per client and is limited in the amount of tweets that can be retrieved as well as historically how far back you can dig up tweets.

##Git it
{%img https://dl.dropbox.com/u/10021156/blog/git.png %}

Probably the simplest and easiest commands you will use when deploying your site to a hosting provider like github, heroku, appharbor. Just remember these few simple commands and you will be able to upload any app anywhere.

{% codeblock lang:bash Git goodness %}
git init //initializes the repo
git add . //adds all files in the current directory to the repo’s changeset
git commit -m “First Commit” //commits all the changes to the repo
git remote add github git@github.com:(username)/(reponame).git
git push github master
You can now import your github repo into www.c9.io and deploy it to cloud foundry.
{% endcodeblock %}

Get the source here: <iframe src="http://ghbtns.com/github-btn.html?user=djohnsonm&repo=Twitter-Feed&type=fork&count=true"
  allowtransparency="true" frameborder="0" scrolling="0" width="95px" height="20px"></iframe>

