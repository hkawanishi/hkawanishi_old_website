---
layout: post
title:  "Week 6 radio buttons in simple form"
date:   2015-12-13 11:07:48 +0000
categories: simple_form radio_buttons
---
I finished my [Flixter][Flixter] app earlier this week and I have some time before starting a group project, so I decided to spend some time on my personal project, “Route 66 Virtual Walk”.  This is an app where users can enter their daily steps and it keeps track of their steps.  The app also converts steps to distance and tracks the user's distance along a virtual version of route 66.


Before week 6, I created a static front page, installed a simple form gem, and created a page for the users to enter the date and their steps.  I also created a summary table where the user's steps and the total distance are displayed.  This week, I tried to set up a user setting page where the user can enter their step length (the distance between a heel of one foot to another when walking), a step length unit (cm or inches), and a distance unit (km or miles).  When creating a database for these inputs, I decided to use integers for all three.  Looking back, I could have used a boolean for the step length unit and the distance unit, but at the time I felt that toggling between 1 and 0 for the unit sounded good.  
I decided to insert a simple form in new.html.erb and edit.html.erb so that the users can create these settings and also update them.  For both step length unit and distance unit inputs, I wanted to use some sort of toggle buttons but initially I didn't know how to do it. 

#### Using f.input for Step Length/Distance Units
If I just use f.input for these like below, they look ugly...  

{% highlight html %}
{% raw %}
<div class="booyah-box col-xs-10 col-xs-offset-1">

  <%= simple_form_for @current_data, :url => walker_usersetting_path(@current_data) do |f| %>
    <h4>Enter Your Step Length (use whole numbers)</h4>
    <%= f.label "Your Step Length:" %>
    <%= f.input :userstride %>

    <br />
    <br />
    <%= f.label "Your Step Length Unit:" %>
    <%= f.input :strideunit %>
    
    <br />
    <br />
    <%= f.label "Your Distance Unit:" %>
    <%= f.input :distanceunit %>

    <%= f.submit 'Create', :class => 'btn btn-primary' %>
  <% end %>
</div>
{% endraw %}
{% endhighlight %}
Above code creates the following form... (What does “Strideunit” = 1 and “Distanceunit” = 1 mean?) 
<figure>
  <a href="/images/usersetting1.png"><img src="/images/usersetting1.png"></a>
</figure>

#### Using checkbox for Step Length/Distance Units

Next I found a way to use a checkbox using f.check_box.  This is what I wrote.

{% highlight html %}
{% raw %}
<div class="booyah-box col-xs-10 col-xs-offset-1">

  <%= simple_form_for @current_data, :url => walker_usersetting_path(@current_data) do |f| %>
    <h4>Enter Your Stride Length (use round number)</h4>
    <%= f.label "Your Step Length:" %>
    <%= f.input :userstride %>
  
    <br />
    <br />
    Default step length unit is in inches.  Uncheck below if you want it in 'cm'.
    <br />
    <%= f.check_box :strideunit %> <br />
    <br />
    <br />
    Default distance is in 'miles'.  Uncheck below if you want it in 'km'.
    <br />
    <%= f.check_box :distanceunit %> <br />
    <br />
    <br />
    <%= f.submit 'Create', :class => 'btn btn-primary' %>
  <% end %>
</div>
{% endraw %}
{% endhighlight %}
This is an improvement but still not as good as I would like it to be (see below).  I still wanted to toggle between the metric system and the imperial system, instead of check/uncheck the box.  
<figure>
  <a href="/images/usersetting2.png"><img src="/images/usersetting2.png"></a>
</figure>

#### Using radio buttons for Step Length/Distance Units
Finally after looking around the web for a while, I found a way to create radio buttons using f.collection_radio_buttons.  And where did I find this information?  It is explained on “simple form gem” GitHub.  Duh!  When am I ever going to learn to read the documentation?    

Here are the code with the radio buttons.  
{% highlight html %}
{% raw %}
<div class="booyah-box col-xs-10 col-xs-offset-1">

  <%= simple_form_for @current_data, :url => walker_usersetting_path(@current_data) do |f| %>
    <h4>Enter Your Step Length (use whole numbers)</h4>
    <%= f.label "Your Step Length:" %>
    <%= f.input :userstride %>
  
    <br />
    <br />
    Step Length Unit:
    <br />
    <%= f.collection_radio_buttons :strideunit, [[1, 'inches'], [0, 'cm']], :first, :last %>
    <br />
    <br />
    Distance Unit:
    <br />
     <%= f.collection_radio_buttons :distanceunit, [[1, 'miles'], [0, 'km']], :first, :last %>
    <br />
    <br />
    <%= f.submit 'Create', :class => 'btn btn-primary' %>
  <% end %>
</div>
{% endraw %}
{% endhighlight %}
This code displays,
<figure>
  <a href="/images/usersetting3.png"><img src="/images/usersetting3.png"></a>
</figure>
I would like to see a bit more space between the buttons,  but this would do for now.  
On to my next adventure, inserting a map to my app!


[Flixter]: http://flixter-hiromi.herokuapp.com/