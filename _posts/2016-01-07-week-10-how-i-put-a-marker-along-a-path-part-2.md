---
layout: post
title:  "Week 10 how I put a marker along a path part 2"
date:   2016-01-07 11:07:48 +0000
categories: maps google_maps google_maps_for_rails
---
Now that I have a csv file which contains the coordinates along Route 66 and distances from the starting point to each coordinate point, I felt I could position a marker on the current location.  I decided to use a [Smarter CSV][Smarter_CSV] which imports csv files as arrays of hashes. Using the Smart CSV gem was easy.  I put the following two statements in my walks_controller.rb

{% highlight ruby %}
  csvfile = File.join(Rails.root, '/app/assets/kml/', 'route66_complete.csv')
  @latlngdata = SmarterCSV.process(csvfile)
{% endhighlight %}

(csvfile is the file I created from a kml file using a simple Ruby file I wrote.  See week 9)
In my walk index.htm.erb file, I first calculate the total distance.

{% highlight html %}
 <% totaldistance = 0 %>
  <% convert_totaldistance = 0 %>
  <% @walks.each do |walk| %>
    <% totaldistance = walk.step * @current_user_setting.step_conversion * @current_user_setting.distance_conversion+ totaldistance %>
  <% end %> 
  <% if @current_user_setting.distanceunit == 1 %>
    <% convert_totaldistance = totaldistance * 1609.34 %>
  <% else %>
    <% convert_totaldistance = totaldistance * 1000 %>  
  <% end %>
{% endhighlight %} 

And then loop through the data in csvfile from a starting position to grab the closest point in csvfile.

{% highlight html %}
<% @latlngdata.each do |point| %>
    <% if point[:dist] > convert_totaldistance %>
      <% @current_lat = point[:lat] %>
      <% @current_long = point[:long] %>
      <% break %>
    <% end %>
  <% end %>
{% endhighlight %} 

I should note that the result is not the closest point.  Rather, it is the closest point which is larger than the convert_totaldistance (which is a user's current position).  In the near future, I plan on change this part of the code to use an algebraic equation (equation of line with two points) so that it can obtain more accurate location for the user.   
Once I got the @current_lat (latitude of the current location) and @current_long (longitude of the current location), I added the following two lines.  (sorry, the variable names are not descriptive... bad coding practice...)

{% highlight html %}
  <div id="foo" data-lat="<%= @current_lat%>"</div>
<div id="foo1" data-long="<%= @current_long%>"</div>
{% endhighlight %} 

and I added the following javascript:

{% highlight html %}
  var lat1 = $("#foo").data("lat");
      var long1 = $("#foo1").data("long");

    function initMap() {
      var map = new google.maps.Map(document.getElementById('map'), {
      zoom: 13,
      center: {lat: lat1, lng: long1}
    });
 // ..... more statements inside initMap function...
{% endhighlight %} 

I should also elaborate on how I added a complete Route 66 kml file.  
I found that I could download the Route 66 kml file for each state from this great site called [Driving Route 66][Driving_Route_66].  I downloaded each kml file, and opened in my Google map.  I deleted alternate paths and any marker along the path.  After this quick editing, I could obtain the file where only the “blue line” of the path was displayed.  I probably could have made this portion more efficient, but I adjusted the ID number manually and ran my Ruby code (from week 9) to create a csv file for each kml file, and concatenate files in the terminal to make one long csv file.  As I said, there probably was a better way to do this but I only had to deal with 8 files.  I did not combine each state kml file.  I just called multiple times from index.html.erb, 

{% highlight html %}
var kmlLayer1 = new google.maps.KmlLayer({
      url: 'https://raw.githubusercontent.com/hkawanishi/route-sixty-six/master/app/assets/kml/route66_illinois_map.kml',
      suppressInfoWindows: true,
      preserveViewport: true,
      map: map
    });

    var kmlLayer2 = new google.maps.KmlLayer({
      url: 'https://raw.githubusercontent.com/hkawanishi/route-sixty-six/master/app/assets/kml/route66_missouri_map.kml',
      suppressInfoWindows: true,
      preserveViewport: true,
      map: map
    });

   // and so on... for each kml file
{% endhighlight %} 

I also cleaned up some of the code related to Google maps in index.html.erb.  In the end, I wasn't sure if I was even using any of the gmap4rails gem functions, but I left the gem just in case.  I also removed a custom map lines because index.html.erb looked really cluttered.  

Now a location marker for a user is displayed along Route 66. 

Here's the map after walking 3.18 miles
<figure>
  <a href="/images/route66-map-1.png"><img src="/images/route66-map-1.png"></a>
</figure>

and  Here's the map after walking 8.07 miles!
<figure>
  <a href="/images/route66-map-2.png"><img src="/images/route66-map-2.png"></a>
</figure>

While I was at it, I also created a “hide” button and a “show” button for the map. 

[Smarter_CSV]: https://github.com/tilo/smarter_csv
[Driving_Route_66]: http://www.drivingroute66.com/route-66-maps/