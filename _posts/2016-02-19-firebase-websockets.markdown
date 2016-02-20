---
layout: post
title:  "Firebase Websockets"
date:   2016-02-19 11:07:48 +0000
categories: websockets 
---
This past Sunday, I finished my group project, which was a collaborative effort to create a game of chess.  Collaborating with other people over the internet was very challenging, but I learned a lot while working on the project.


In this project, we wrote code in such a way that one player can create a new game and another player can join the game.  We were able to implement most of the chess piece move validations but we had a problem where browsers would not refresh when a game piece is moved.


For example, one player would move a pawn in his/her browser, but the other player wouldn't see this movement until he/she manually refreshed their browser.  We asked our mentor for advice on how to fix this problem. He suggested two solutions to this problem: we could write polling code in javascript or we could implement websockets.  


While deciding between polling and websockets, I just happened to see [Firebase][firebase] websockets mentioned in the “Job Prep” section (access only to their students) on the [Firehose Project][firehoseproject] website.  I had never heard of it before but I clicked on it and read about it. It turned out that this was exactly what I was looking for.  There was a good interactive tutorial on the website, and I was able to grasp the concept pretty quickly. 
I also searched for some examples of javascript polling code but it looked more difficult than implementing websockets so I decided to work to implement the websockets.  

And since this was one of the last things I worked on for our group project, I am going to write about this.

This was what I did to make it work:

* I signed up for [Firebase][firebase] account.  A basic account is free.  At this time I only needed this account for my group project so the free account was sufficient.


* I used [Firebase Ruby Gem][firebaserubygem].
  I put:

  {% highlight ruby %}
  `gem 'firebase'`
  {% endhighlight %}

    in Gemfile and `bundle install`

* I tried to follow the documentation in [Firebase Ruby Gem][firebaserubygem].  I read the document a couple of times and played around with some code.  I understood that the first thing I needed to do was to point to my Firebase account and then push my data into the account.  I thought about where to add the code and decided that the best place to add it was inside the  “move” method, right after a game piece was moved (inside games_controller.rb).
I put the following code in just like it said in the documentation:


{% highlight ruby %}
base_uri = 'https://<your-firebase>.firebaseio.com/'   #note: For <your-firebase> part, I inserted my Firebase account link
firebase = Firebase::Client.new(base_uri)
{% endhighlight %}

  Under these two lines, I also added the following statements:
{% highlight ruby %}
response = firebase.push(game_path, @game_piece, :created => Firebase::ServerValue::TIMESTAMP)
response.success? # => true
{% endhighlight %}

  After I added these lines and tried moving a chess piece and then looked at my Firebase account, I saw the data was pushed successfully.

  <figure>
    <a href="/images/firebase_sample1.png"><img src="/images/firebase_sample1.png"></a>
  </figure>

  The good news was that I was able to push the data.  But then, I was totally confused.  All of the examples in the Firebase tutorials are on pushing new data and doing something with the data (for example, adding and displaying the new data on the screen.)   But all I needed to do was to refresh the browser.  How was I supposed to use the data to refresh the browser?


  I decided to get help from our nice [Firehose Project][firehoseproject] community.  I hoped somebody could give me some suggestions.  Within a few hours of posting my question, I was in a private chat with a very nice person named Aaron who had experience with Firebase.  I showed him what I had, and he suggested that I could just use a timestamp to refresh the browser, and told me that it isn't necessary to pass chess game piece variables.  Basically I could just send data for the time when the piece was moved to my Firebase account, and then compare that to the last time the browser was refreshed.  I could write the code to refresh browsers, if (the time when the game piece was moved) > (the browser time).  


  I changed the above response variable so that only a game ID and a timestamp were passed to firebase:
  {% highlight ruby %}
  response = firebase.set("#{@game.id}", :created => Firebase::ServerValue::TIMESTAMP)
  {% endhighlight %}

  After this change, my account only had “game id” and “timestamp” as shown below:

  <figure>
    <a href="/images/firebase_sample2.png"><img src="/images/firebase_sample2.png"></a>
  </figure>

* I inserted the Time.now statement inside games/show.html.erb (the game board and the game pieces are displayed in this file).  This stores the time when the browser refreshes.  

  {% highlight html %}
    <div id="timestamp" data-time-stamp="<%= (Time.now.to_f * 1000).to_i %>"></div>
  {% endhighlight %}

*  I then wrote javascript to grab the data from Firebase and compare it to the time stamp in games/show.html.erb
  

{% highlight html %}
var firebase_path = 'https://<your-firebase>.firebaseio.com/'  + '<%=@game.id%>' + "/";   //Inside this account, I have game id and then time stamp.  That is why I had to add @game.id under 'https://<your-firebase>.firebaseio.com/'  path (See figure above.  For example, "324" is game_id)

var myDataRef = new Firebase(firebase_path);

var load_time_stamp = $('#timestamp').data('time-stamp');  //id=”timestamp” is written in the same file in the previous step

// the statement below grabs the timestamp "created" from firebase database.
myDataRef.on('value', function(snapshot) {
  data = snapshot.val();
  var time_fb = data.created;   //data.created because I used :created => to add Timestamp) 

  if (time_fb > load_time_stamp) {  //compare time stamps in browser vs in firebase
    location.reload();   //this refreshes the browser
  }
});
{% endhighlight %}

That is how I implemented the websockets for our group project.  I am sure there is a better way to do this but it worked for our project so I was very pleased with the outcome.

[firebase]: https://www.firebase.com/
[firehoseproject]: http://www.thefirehoseproject.com/
[firebaserubygem]: https://github.com/oscardelben/firebase-ruby