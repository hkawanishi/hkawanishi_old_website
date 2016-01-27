---
layout: post
title:  "Week 12 understanding crud operations part 1: from index to update "
date:   2016-01-22 11:07:48 +0000
categories: CRUD
---

I had some trouble understanding how CRUD works in Rails.  I understand what CRUD does but when I wanted my web app to go to a certain controller from a view file, I had a hard time making it work.  Furthermore, when I somehow managed to make it work, I didn't know why it worked.  

When I searched about CRUD on the web, I found a page that explains the concept.  CRUD is an acronym for the four operations it performs on data: “C”reate, “R”ead, “U”pdate, and “D”estroy.  Active Record on Rails supposedly is nice and streamlined to help us automatically create and post and manipulate data.  Almost every time I search for a Rails tutorial, I find instructions about creating some sort of a blog post app where you can take advantage of CRUD.  In the tutorial, you can create a post (create), display a single post or several posts (read), edit posts (update) and delete posts (destroy).  This kind of tutorial is great, but when I started doing a different kind of project which didn't fit to this process, I struggled with calling a particular controller action.  I tried to find an answer on the web, but I couldn't find a good explanation.  Maybe there were answers written somewhere but it just didn't sink into my brain.  As I wrote earlier, I sometimes managed to make it work but I didn't know why it worked.  I was frustrated.  

I decided to ask about this to my mentor, and we had a long conversation about it.  He was very patient and explained a lot of things to me I couldn't find on the web.  After I talked to him, I think a lot of concepts got clearer to me.  In this blog post and the next one, I am going to try to write about what I learned.

### Create Action
As I said above, if you have completed your first Rails project, it was most likely a blog type project and your project probably has a function to create a new post.  You probably learned that the “C” part of “CRUD” operation means “Create” and it involves a two-step process using the “new” and “create” actions.  “New” would display a web form in which data will be entered, and when you submit the form, it goes to the “create” action.  In other words, “New” is used to submit the form, and “create” is used to process the form.


<figure>
  <a href="/images/crud_table.png"><img src="/images/crud_table.png"></a>
</figure>


The “R” part of “CRUD” means “Read”.  This also has two actions, but it doesn't have a “submit form” and “process form” pair like the other functions.  Instead, “index” returns a list of records, and “show” returns a single record.  For example, if you have a blog site, “index” will display all blog entries, and “show” will display a single blog entry.   
Rails also uses the RESTful (Representation State Transfer) as default routes.  REST uses HTTP requests to get, put/patch, post, and delete data.  When you type “rake routes” on your terminal, you can see the defined routes for all standard RESTful actions. 

<figure>
  <a href="/images/sample-routes.png"><img src="/images/sample-routes.png"></a>
</figure>
  (from [Ruby on Rails Guide][rubyrailguide])


From the picture above, you can see what the action verb is, what the corresponding url is, and which controller they link to.  


I am involved in a small game project, which has the following line in index.html.erb (using simple_form gem).  For this app, a user can first create a game from index.html.erb, and then after the game is created, it goes back to the index page again and waits until there is an opponent. 

{% highlight html %} 
<%= simple_form_for Game.new, :url => games_path do |f| %>
{% endhighlight %}

When I type “rake routes” for this project, I see:

{% highlight html %} 
games   GET     /games(.:format)      games#index                             
        POST    /games(.:format)      games#create                           
new_game GET    /games/new(.:format)  games#new                          
{% endhighlight %}

From the line above, you see that it is creating a new game using the Game.new statement, and you notice that the url is pointing to “games_path”.  If you look at the above results of “rake routes”, you see that the left most column has helper routes and it says that “games_path” points to the “index” controller.  Then, you might think, “Wait a minute, index uses the GET action.  GET is used to display stuff.  Whatever happened to POST?  What happened to new/create pair?”  
If I go to this index page and right click to “Inspect” it in my browser and examine how this new game is created, I see the following line:

{% highlight html %}
<form accept-charset="UTF-8" action="/games" class="simple_form new_game" id="new_game" method="post" novalidate="novalidate">
{% endhighlight %}

This line tells me that it is using the “post” method.  Even though it is pointing to “games_path” and “rake routes” said games_path uses the “get” action, it is actually using POST instead of the GET action, because it just “created” a new game.  If I look at the “create” method in games_controller.rb, it has the following statements:


{% highlight ruby %}
def create
  @game = Game.create(game_params)
  redirect_to games_path
end
{% endhighlight %}

Therefore, Rails goes to create new data using Game.new, and go to a “create” controller action, and then go to “index” controller action.  
To verify this, I looked at the terminal, when I created a game I saw:
{% highlight html %} 
Processing by GamesController#create as HTML
{% endhighlight %}
and some lines after that, I saw the redirect message to /games (index.html.erb)
{% highlight html %} 
Redirected to http://localhost:3030/games
{% endhighlight %}


### Update Action

The “U” part of “CRUD” is for “Update”.  “Update” is similar to “Create”; it is a two-step process.  The difference is that when we want to “Update”, it updates the existing data instead of creating new data.  “Edit” displays a form, and “Update” processes the form. 
In the same index.html.erb for a game project, there is another “simple_form_for” statement. 
{% highlight html %}  
  <%= simple_form_for game do |f| %>
{% endhighlight %}

(note: the reason it is “game” instead of “@game” in above statement is that there is a loop before this statement and it contains the line like the following.  (...) is omitted here for brevity.
  <% @game.where(...).find_each do |game| %>
)


This statement is called after a user creates a new game, and an opponent joins the game.  The question is that when this statement is called, which controller action will be called next?
My mentor suggested that I should look at the source code using “inspect” in my browser again.  When I looked at, I first saw:

{% highlight html %}
<form id="edit_game_271" class="simple_form edit_game" novalidate="novalidate" method="post" action="/games/271" accept-charset="UTF-8">
{% endhighlight %}

There are a couple of interesting things in here.  First of all, “form id” is now “edit_game_271” (271 comes from game_id, which we set in database).  One of the things I was really confused about CRUD before I talked with my mentor was that I thought I have to have an “edit” controller and an “update” controller together.  However, Rails decides if there is a form and the record is persisted, it uses the “edit” action regardless where it is calling from.  And if there is no data, it uses the “new” action. (note: my mentor used the word “persisted” when he was explaining to me about this.  I had never heard of the word “persisted” with data before, and when I asked him, he explained that “data is persisted” means “the data already exist”.  I thought it is cool to use this word here...)  

The second interesting thing is that it said that the method is “post”.  But when I “edit” data, shouldn't it go to the “update” controller?  
The answer is “yes”.  And this was another confusing part.  My mentor then pointed out to me that I should “inspect” more.  
As it turns out, under <form id="edit_game_271" ... , there is a div style statement and then there is a hidden _method statement.

{% highlight html %}
<div style="display:none">
<input type="hidden" value="✓" name="utf8">
<input type="hidden" value="patch" name="_method">
{% endhighlight %}

As you see from this hidden _method statement above, it is actually calling the “patch” action instead of the “post” action.  Thus, the statement:

{% highlight html %}
  <% @game.where(...).find_each do |game| %>
{% endhighlight %}

in index.html.erb uses the “edit” action and then calls the “update” action. 
Sometimes, it is not clear which action follows which just by looking at a html.erb file, so it is important to think about action steps.  And I think it is good practice to check a message in a terminal and inspect the file often to make sure Rails is doing what you want it to do.  


More discussion on CRUD operations continue next time...

[rubyrailguide]: http://guides.rubyonrails.org/getting_started.html