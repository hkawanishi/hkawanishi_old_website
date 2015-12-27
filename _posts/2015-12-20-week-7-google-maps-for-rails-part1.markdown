---
layout: post
title:  "Week 7 google maps for rails part 1"
date:   2015-12-20 11:07:48 +0000
categories: maps google_maps google_maps_for_rails
---
In week 7, I continued working on my “Route 66 Virtual Walk” app.  In previous weeks, I  implemented the user setting page and the walk summary page.  Next, I needed to install a map.  In [Nomster][Nomster], I used a gem called [Geocoder][Geocoder].  Geocoder was great in Nomster; all I had to do is to provide an address, and it automatically displayed a map with a location pin in the given address.  For “Route 66 Virtual Walk”, I needed something besides Geocoder because I needed to (1) create and display a path of Route 66 between Illinois to California, and (2) calculate where the user is along with that path using the step inputs. 

I decided to work on (1) first.  I thought it would be nice if I could display map of the US and then add a line indicating Route 66 on top of the map.  After researching for a while, I thought that [Google Maps for Rails][gmap4rails] might help me accomplish this task.  I chose this gem because the documentation said it can display KML (“Keyhole Markup Language”) files.  Using Google Map, I can easily create a path from point A to point B and save it in a KML file.  I wasn't sure how easy it would be to display a map with KML overlay but I thought it was worth trying.  Here is how I installed the gem and what sort of maps I was able to create.  

I created the “location” model and controller.  The migration file contains:
{% highlight ruby %}
t.string :address
t.float :latitude
t.float :longitude
t.integer :location_id
t.integer :user_id
{% endhighlight %}

I first installed the Geocoder gem just like I did when I was working on the Nomster project.  Then, I put the following line in my gemfile and `bundle install`.
{% highlight ruby %}
  gem `gmaps4rails`
{% endhighlight %}

In my view/walker/locations/index.html.erb file, I put the following statements as described in “Google Map for Rails” documentation.
{% highlight html %}
{% raw %}
<div style='width: 800px;'>
  <div id="map" style='width: 800px; height: 400px;'></div>
</div>
<script src="//maps.google.com/maps/api/js?v=3.18&sensor=false&client=&key=&libraries=geometry&language=&hl=&region="></script> 
<script src="//google-maps-utility-library-v3.googlecode.com/svn/tags/markerclustererplus/2.0.14/src/markerclusterer_packed.js"></script>
<script src='//google-maps-utility-library-v3.googlecode.com/svn/tags/infobox/1.1.9/src/infobox_packed.js' type='text/javascript'></script> 
{% endraw %}
{% endhighlight %}

The next part of the documentation said that I should install underscore.js, so I added
{% highlight ruby %}
  gem 'underscore-rails'
{% endhighlight %}
in my gemfile and `bundle install` again.

Then I created a new file called underscore.js and saved it under vender/assets/javascripts/.   I went to [underscorejs.org][underscorejs.org] and clicked the “[production version (1.8.3)][underscore-min.js]” link, selected all the text in that page, and pasted all the text into underscore.js.   (See [“Google maps for Rails v2” youtube video][gmap4rails_youtube]).  I then added the following lines in app/assets/javascripts/application.js

{% highlight ruby %}
//= require underscore
//= require gmaps/google
{% endhighlight %}

And I inserted the snip of the code which was provided in the documentation.  I used latitude = 41.87839 and longitude = -87.61717 which is close to the starting point of my Route 66.

{% highlight html %}
{% raw %}
<script type="text/javascript">
    
  handler = Gmaps.build('Google');
  handler.buildMap({ provider: {}, internal: {id: 'map'}}, function(){
    markers = handler.addMarkers([
    {
      "lat": 41.87839,
      "lng": -87.61717,
      "picture": {
        "url": "http://people.mozilla.com/~faaborg/files/shiretoko/firefoxIcon/firefox-32.png",
        "width":  32,
        "height": 32
      },
      "infowindow": "hello!"
    }
    ]);
    handler.bounds.extendWith(markers);
    handler.fitMapToBounds();
  });
</script>
{% endraw %}
{% endhighlight %}

This displays:
<figure>
  <a href="/images/gmap4rails1.png"><img src="/images/gmap4rails1.png"></a>
</figure>

It looks like the map is working!  There are examples of map overlays on [Gmap4Rails][Gmap4RailsExamples] which include Javascript and HTML code.  I wanted to see if I could include a circle in my map.

I used some example code in [Gmap4Rails][Gmap4RailsExamples] under “Multiple Overlays” for drawing a circle.  My code looked like:

{% highlight html %}
{% raw %}
<div style='width: 800px;'>
  <div id="map" style='width: 800px; height: 400px;'></div>
</div>

<script type="text/javascript">    
  handler = Gmaps.build('Google');
  handler.buildMap({ internal: {id: 'map'}}, function(){
    var circles = handler.addCircles(
    [{ lat: 41.8783968, lng: -87.6171476, radius: 50 }],
    { strokeColor: '#FF0000'}
    );
    handler.bounds.extendWith(circles);
    handler.fitMapToBounds();
  });
</script>

{% endraw %}
{% endhighlight %}

This code produced a circle on the same map.
<figure>
  <a href="/images/gmap4rails2.png"><img src="/images/gmap4rails2.png"></a>
</figure>

The same site also includes example code for a custom style map.  It said that all I had to do is to create `var mapStyle = []` and then add code from [Snazzy Maps][snazzymaps].  For example, I could add the “[Unsaturated Browns][unsaturated_browns_map]” map.   My code looked like:

{% highlight html %}
{% raw %}
<script type="text/javascript">
  var mapStyle = [
    {
        "elementType": "geometry",
        "stylers": [
            {
                "hue": "#ff4400"
            },
            {
                "saturation": -68
            },
            {
                "lightness": -4
            },
            {
                "gamma": 0.72
            }
        ]
    },
    {
        "featureType": "road",
        "elementType": "labels.icon"
    },
    {
        "featureType": "landscape.man_made",
        "elementType": "geometry",
        "stylers": [
            {
                "hue": "#0077ff"
            },
            {
                "gamma": 3.1
            }
        ]
    },
    {
        "featureType": "water",
        "stylers": [
            {
                "hue": "#00ccff"
            },
            {
                "gamma": 0.44
            },
            {
                "saturation": -33
            }
        ]
    },
    {
        "featureType": "poi.park",
        "stylers": [
            {
                "hue": "#44ff00"
            },
            {
                "saturation": -23
            }
        ]
    },
    {
        "featureType": "water",
        "elementType": "labels.text.fill",
        "stylers": [
            {
                "hue": "#007fff"
            },
            {
                "gamma": 0.77
            },
            {
                "saturation": 65
            },
            {
                "lightness": 99
            }
        ]
    },
    {
        "featureType": "water",
        "elementType": "labels.text.stroke",
        "stylers": [
            {
                "gamma": 0.11
            },
            {
                "weight": 5.6
            },
            {
                "saturation": 99
            },
            {
                "hue": "#0091ff"
            },
            {
                "lightness": -86
            }
        ]
    },
    {
        "featureType": "transit.line",
        "elementType": "geometry",
        "stylers": [
            {
                "lightness": -48
            },
            {
                "hue": "#ff5e00"
            },
            {
                "gamma": 1.2
            },
            {
                "saturation": -23
            }
        ]
    },
    {
        "featureType": "transit",
        "elementType": "labels.text.stroke",
        "stylers": [
            {
                "saturation": -64
            },
            {
                "hue": "#ff9100"
            },
            {
                "lightness": 16
            },
            {
                "gamma": 0.47
            },
            {
                "weight": 2.7
            }
        ]
    }
]
    
  handler = Gmaps.build('Google');
  handler.buildMap({ provider: {
      zoom: 15,
      center: new google.maps.LatLng(41.8783968,-87.6171476),
      mapTypeId: google.maps.MapTypeId.ROADMAP,
      styles: mapStyle
    }, internal: {id: 'map'}}, function(){

    var circles = handler.addCircles(
    [{ lat: 41.8783968, lng: -87.6171476, radius: 50 }],
    { strokeColor: '#FF0000'}
    );

    markers = handler.addMarkers(<%=raw @hash.to_json %>);
    handler.bounds.extendWith(markers);
    handler.bounds.extendWith(circles);
    handler.fitMapToBounds();
    });
</script>

{% endraw %}
{% endhighlight %}

This changed my map to :
<figure>
  <a href="/images/gmap4rails3.png"><img src="/images/gmap4rails3.png"></a>
</figure>

This map looks cooler than the original one!

This was how I installed [Google Map for Rails][gmap4rails] and played with it to see what sort of overlays I could do.  I will be writing about the map with KML overlay (which was the original intention) next.  


[Nomster]: http://nomster-by-hiromi.herokuapp.com/
[Geocoder]: https://github.com/alexreisner/geocoder
[gmap4rails]: https://github.com/apneadiving/Google-Maps-for-Rails
[underscorejs.org]: http://underscorejs.org
[underscore-min.js]: http://underscorejs.org/underscore-min.js
[gmap4rails_youtube]: https://www.youtube.com/watch?v=R0l-7en3dUw&feature=youtu.be
[Gmap4RailsExamples]: http://apneadiving.github.io/
[snazzymaps]: https://snazzymaps.com/
[unsaturated_browns_map]: https://snazzymaps.com/style/70/unsaturated-browns
