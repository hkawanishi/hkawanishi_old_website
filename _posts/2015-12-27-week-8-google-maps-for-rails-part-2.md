---
layout: post
title:  "Week 8 google maps for rails part 2"
date:   2015-12-27 11:07:48 +0000
categories: maps google_maps google_maps_for_rails
---
After I played around with Google Maps for Rails (see Week 7 entry), I tried to implement a KML overlay.  I already wrote about this in the Week 7 entry, but what I wanted to do was to have some sort of line on my map indicating the “path” of Route 66 from Illinois to California.  The idea was to first display the line of the path and then put a marker on the map to show how far a user has walked along the path.  

I have never heard of a KML file format until very recently, but when I was researching how to draw a line on my map, I found many sites talking about this format.  Basically, KML is a file format which draws lines, markers, and shapes on maps using Google Earth or Google Map.  KML uses XML notation, so it can be opened using a text editor and the content of the file can be examined.  

I watched this YouTube video, [“Using Google Maps for geocoding addresses and creating KML”][Using Google Maps for geocoding addresses and creating KML] and I learned how to draw a line and how to download the file as a kml format.  Rather than drawing all of Route 66 immediately, I decided to test the program by creating a short path.  I created a path from Chicago to Hodgikins, IL using [historic66][historic66] website as a reference and saved it as a KML file.  

According to some example code shown in [Gmap4Rails][Gmap4RailsExamples], I need to add javascript code like this:

{% highlight html %}
  var handler = Gmaps.build('Google');
  handler.buildMap({ internal: {id: 'with_kml'}}, function(){
  var kmls = handler.addKml(
    { url: "http://gmaps-samples.googlecode.com/svn/trunk/ggeoxml/cta.kml" }
  );
});
{% endhighlight %}

and add 

{% highlight html %}
<div style='width: 800px;'>
  <div id="kml" style='width: 800px; height: 400px;'></div>
</div>
{% endhighlight %}

in my html.erb file.  I first inserted the code above to see if it worked.  It did!  I saw some colorful lines on my map.  Next, I put my kml inside an app/assets folder to see if it worked.  It didn't.  The blue line I created was not displayed on my map.  I played around with the name of the file path to see if I could make this work.  But it didn't.  I tried to narrow down if the problem was the file format or a path, so I downloaded the above cta.kml file to my machine, and put it in my local path.  The lines did not show up on my map.  That means that the url: in the above statements did not accept a local path.  
During a meeting with my mentor, I asked what I could do to solve this problem.  My first thought was to put the file somewhere like Amazon S3, but my mentor gave me a very simple solution.  He suggested that I could do this using my gitHub repo! I have my kml file in the app/assets/kml folder, and I already pushed the file in my gitHub.  Actually I tried to point to my kml file in gitHub before but it didn't work, so I didn't try again.  But apparently I wasn't doing this right.  What I needed to do was to go to my gitHub, click my kml file, and then I need to click “raw” to get the correct path.  Once I put that path in my walk index.html.erb file, it worked!

<figure>
  <a href="/images/kml-1-try.png"><img src="/images/kml-1-try.png"></a>
</figure>

[Using Google Maps for geocoding addresses and creating KML]: https://youtu.be/foB8TjpB4K8
[historic66]: http://www.historic66.com
[Gmap4RailsExamples]: http://apneadiving.github.io/