---
layout: post
title:  "Week 9 how I put a marker along a path part 1"
date:   2015-12-31 11:07:48 +0000
categories: maps google_maps google_maps_for_rails
---
Now that I know how to create a blue path to indicate Route 66 on the map, my next challenge is to figure out how to calculate where the user is along Route 66 and put a location marker on top of it.  



I know my available inputs are:

* the total distance a user has traveled (calculated from user's step inputs and each step length)
* kml file which include the information of a line along Route 66.
* starting location (Chicago)


From here I want to get:

* Latitude and Longitude of the user's location along Route 66.


I looked around in the Google Maps documentation to see if I could find a way to do it.  I found a way to create a marker if I know the appropriate latitude and longitude.  I also found a way to calculate the distance between point A and point B.  However, I couldn't find the solution to what I wanted (I know the distance and want to get the location).  

I brought this subject to our weekly programming camp office hour.  A couple of people mentioned PostGIS which I had never heard of until then.  Also, Ken mentioned that a former student named Jason had done similar to my project and gave me a link of his project in gitHub.  I cloned Jason's project and ran it in my local machine.  The project looked very cool but I didn't see where the current location is calculated.  
I didn't know anything about PostGIS.  I signed up for one of GIS classes in Udemy and also watched another GIS class in Lynda.com to help me understand GIS.  I spent two days learning GIS and it was very interesting.  However, I still couldn't find a quick answer to my problem. 


I thought about this for a while.  And then, I started thinking that maybe the problem was that I was spending too much time trying to find a quick and easy answer.  Instead of looking for a convenient Gem, I decided to focus on a kml file.  I found that the coordinates for lines are included in kml file.  Those coordinates are between <coordinates> and </coordinates> tags, and they have repeated numbers which signify longitude, latitude, and altitude.  

<figure>
  <a href="/images/screen-shot-kml-file.png"><img src="/images/screen-shot-kml-file.png"></a>
</figure>

For example, in line 15 of the above picture, the location of the first point is (-87.61717 [longitude], 41.87839 [latitude]) with 0.0 altitude.  The location of the second point is (-87.61743 [longitude], 41.87839[latitude]) with 0.0 altitude.  The distance between the points are not equally distributed.  I am not sure how these points were created since all I did was to draw a line using Google Map (see week 8), but I knew these points were at least located along the line.  

My thought was to grab these numbers and create a separate file with a table which contains:


  {% raw %}
    ID number, Longitude, Latitude, Altitude, Distance Between Points
  {% endraw %}


I decided that the ID number would start from 0.  The reason I included “ID number” was that I thought it would be handy to have it later when I would be searching for position.  Longitude, Latitude, and Altitude were obtained from the kml file.  Distance Between (coordinate) Points would be calculated according to Haversine formula which was discussed [here][haversine-formula]


I decided to write some simple Ruby code to do this task.  This Ruby file would (1) grab the numbers between <coordinates> and </coordinates> inside <LineString> (under <tessellate> tags), (2) write these coordinate points in order, (3) add an ID number, and (4) calculate the distance.  The new file would be in csv format with commas.  
Here's the example of the first four lines of csv file which was created from kml file.


  {% raw %}
    id,long,lat,alt,dist
    0,-87.61729000000001,41.87839,0.0,0.0
    1,-87.61759,41.87839,0.0,24.837501018796686
    2,-87.6188,41.87836,0.0,125.07097049154589
  {% endraw %}


I should note that I didn't add “altitude” in the distance calculation, but I decided to include it in case I wanted to implement more accurate calculation in the future.  (also note that the distance was calculated in meters).

I didn't see the need to include this Ruby code in the Rails model file, since this is just a data file of Route 66 and it did not need to be modified while the web app is running.  Once this Ruby file created a csv file from a kml file, the purpose of this code ends there. 

Now all I had to do was to use this csv file information and obtain the user's location.  I will write about it next time...

[haversine-formula]: http://stackoverflow.com/questions/27928/calculate-distance-between-two-latitude-longitude-points-haversine-formula