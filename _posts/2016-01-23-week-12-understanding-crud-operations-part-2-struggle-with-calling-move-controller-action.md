---
layout: post
title:  "Week 12 understanding crud operations part 2: struggle with calling Move controller action"
date:   2016-01-22 11:07:48 +0000
categories: CRUD
---
This is part 2 of what I learned about CRUD operations.  For more discussions, please see part 1.  


In the game project I am involved in, we wanted to “select” a game piece on a board and then “move” the piece.  It seemed simple, but we had a such a struggle trying to tell a view file to go to a move controller.  


In part 1, I described how a game is created by one user and then the game is updated after another player joined in.  After that, it goes to a “show” page (show.html.erb) to display all of  the game pieces and the game board.  Our intent was that we should be able to select a piece from show.html.erb, go to the “select” controller action, and display the board with the selected piece highlighted in select.html.erb.  From there, a user can select the destination of the game piece, go to the “move” controller action, and then display in show.html.erb the board and game pieces with the selected piece moved to a new position.  In short, we wanted to do:


  show.html.erb (all game pieces and a board are displayed)-> select a game piece -> select controller -> select.html.erb (the selected piece is highlighted) -> select a destination for the selected game piece -> move controller (database update) -> show.html.erb (all game pieces and the board are displayed; one game piece is in a new position)


To achieve this process, we added the following statements in our routes.rb for “select”.
{% highlight html %} 
  get 'games/:id/:game_piece_id/:x/:y', to: 'games#select', as: 'select'
{% endhighlight %}

We created select.html.erb, and we want to select a destination for the game piece from this file, so the “get” action was used for “games#select”.  We decided to include “game id,” “game piece id,” “current x location,” and “current y location” in the path, so that we can pass these variables.  I should note that this is still an early stage of the project, and to select a game piece and a destination of the game piece, a user has to click the corresponding “links”.  
In show.html.erb, we have a statement similar to the following:
{% highlight html %} 
<%= link_to "some_game_piece", select_path(:game_piece_id => game_piece, :x => game_piece.x, :y => game_piece.y) %>
{% endhighlight %}
As you see, if you click the “some_game_piece” link, it goes to the “select” controller.  “select_path” is a helper path name.  When I type “rake routes”, I see:

{% highlight html %} 
  select GET    /games/:id/:game_piece_id/:x/:y(.:format)   games#select
{% endhighlight %}
And if I put the cursor on the link of one of the pieces (in above statement, this link is defined as a “some_game_piece”), I see the intended link; for example, it shows ../../279/1002/0/7 (where 279 is the game id, 1002 is the game piece id, 0 is the x-coordinate, and 7 is the y-coordinate).  So after selecting the link of a game piece in show.html.erb, it goes to the select controller action and then goes to select.html.erb with the piece highlighted just like we intended.  Success.  


We decided to do the same thing for a “move” controller.  However, we do not need to display the data in move.html.erb.  A select controller action can send a destination location to the move controller action, and we only need to process the data in “move”.  After updating the data in the move controller action, it goes to show.html.erb and display the board with a piece moved.
We put the following statement in routes.rb

{% highlight ruby %}
  post 'games/:id/:game_piece_id/:new_x/:new_y', to: 'games#move', as: 'move'
{% endhighlight %}
This is very similar to “games#select”.  Except it uses “new_x” and “new_y” (the new destination x and y coordinates, respectively), and uses “post” instead of “get” because we don't need to get the data; we only need to process it.  
In select.html.erb, we inserted the following statement.
{% highlight html %} 
<%= link_to 'next move', move_path(@game, :game_piece_id => @game_piece.id, :new_x => col, :new_y => index) %>
{% endhighlight %}

Also, when I type “rake routes”, I see:
{% highlight html %} 
  move POST   /games/:id/:game_piece_id/:new_x/:new_y(.:format)    games#move
{% endhighlight %}

Everything looked good to me.  We wanted the “move” controller to have a “post” action, so we set “move_path” in select.html.erb.  It looks like it should work.


Except it doesn't work.  


So why is it not working?  It is because the above statement has a “link_to”.  When the controller is called by a “link,” Rails assume it is calling a “GET” action.  Yes, it says “move_path” but it completely ignores it.  When I click the “next move” link, I see there is a message in terminal says the following:

{% highlight html %} 
  Started GET "/games/279/1002/0/5" for ....
{% endhighlight %}

For Rails, it seems that it is more important to have the correct “action” than the path name.  Rails searches for /games/279/1002/0/5... for a “get” action.  It ignores the move path because move path contains a “post” action.  On the other hand, “select” has a “get” action, so it goes back to a select controller.  
To make sure it goes to the “move” controller, I needed to add `[:method => :post]` flag in the statement:

{% highlight html %} 
<%= link_to 'next move', move_path(@game, :game_piece_id => @game_piece.id, :new_x => col, :new_y => index),:method => :post %>
{% endhighlight %}

This makes Rails to go look for a “post” action instead of a “get” action so that move_path is found.



In summary, these are what I learned about CRUD operations and RESTful actions:

* The PATH and METHOD are the 2 variables for how Rails routes from web request to controller action.
* It is important to think about which “action” comes next (“verb” is important).
* Don't rely on a helper path.
* In general, a “link” points to a “get” action, and a “form” points to a “post” action.  If you want a different action to follow, you need to tell Rails.    
* If the data is persisted, it uses the “edit/update” action, and if data is newly created, it uses the “new/create” action regardless of how data is created.
* Check your terminal and check your file using “inspect” in your browser.  
