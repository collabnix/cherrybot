# Stream Data Over PubNub

If you haven’t already done so, [sign up for a free PubNub account](https://admin.pubnub.com/) before you begin this step.



## Change directory 

Change directory into the examples directory containing the gps_simpletest.py file and install the PubNub Python SDK.

```
pip3 install pubnub
```

## Import PubNub Package

```
import pubnub
from pubnub.pnconfiguration import PNConfiguration
from pubnub.pubnub import PubNub
from pubnub.callbacks import SubscribeCallback
from pubnub.enums import PNOperationType, PNStatusCategory
```

## Configure a PubNub instance with your publish/subscribe Keys

```
pnconfig = PNConfiguration()
pnconfig.subscribe_key = "YOUR SUBSCRIBE KEY"
pnconfig.publish_key = "YOUR PUBLISH KEY"
pnconfig.ssl = False
pubnub = PubNub(pnconfig)
```

Then to publish, place a publishing callback somewhere near the beginning of your code. You can write whatever you want for the callback, but we’ll leave it blank as we don’t really need it for now.

```
def publish_callback(result, status):
    pass
    # Handle PNPublishResult and PNStatus
```


Here is where you decide what data you want to publish. Since we are building just a simple GPS tracking device, we’re just going to be dealing with the latitude and longitude coordinates.

When you want to publish multiple variables in one JSON, you must create a dictionary like so:

```
dictionary = {"DATA 1 NAME": gps.DATA1, "DATA 2 NAME": gps.DATA2}
```

So in our case we would write:

```
dictionary = {"latitude": gps.latitude, "longitude": gps.longitude}
```
And then to publish that data, you would format the dictionary like this:

```
pubnub.publish().channel("CHANNEL").message(dictionary).pn_async(publish_callback)
```

It is best to place the dictionary and publishing lines within the “if gps.DATA is not none” to avoid any program failures.


## Visualize your GPS Data with Google Maps

It's time to visualize our GPS data in a way that humans can understand.

We’re just going to create a small HTML page that will grab GPS data from our PubNub channel and graph the data with a geolocation API.

## Google Maps API

The Google Maps API is a universal tool that is not only one of the cheaper APIs for a greater amount of API calls but also has a rich and expansive toolset for developers. The GPS data is not only more accurate than most other APIs, but also has extensive tools such as “ETA” that uses Google’s geographical terrain data.

So if you ever want to build a serious GPS tracking app with PubNub, Google Maps is the way to go.

Image result for google maps api with marker

You’ll first need to get a Google Maps API Key

Once that’s done, create an .html file and copy-paste the code below (explanation of the code is below as well).

```
<!DOCTYPE html>
<html>
  <head>
    <title>Simple Map</title>
    <meta name="viewport" content="initial-scale=1.0">
    <meta charset="utf-8">
    <style>
      /* Always set the map height explicitly to define the size of the div
       * element that contains the map. */
      #map {
        height: 100%;
      }
      /* Optional: Makes the sample page fill the window. */
      html, body {
        height: 100%;
        margin: 0;
        padding: 0;
      }
    </style>
    <script src="https://cdn.pubnub.com/sdk/javascript/pubnub.4.23.0.js"></script>
  </head>
  <body>
    <div id="map"></div>
    <script>
  // the smooth zoom function
  function smoothZoom (map, max, cnt) {
      if (cnt >= max) {
          return;
      }
      else {
          z = google.maps.event.addListener(map, 'zoom_changed', function(event){
              google.maps.event.removeListener(z);
              smoothZoom(map, max, cnt + 1);
          });
          setTimeout(function(){map.setZoom(cnt)}, 80); // 80ms is what I found to work well on my system -- it might not work well on all systems
      }
  } 
    var pubnub = new PubNub({
    subscribeKey: "YOUR SUBSCRIBE KEY",
    ssl: true
  });  
  var longitude = 30.5;
  var latitude = 50.5;
  pubnub.addListener({
      message: function(m) {
          // handle message
          var channelName = m.channel; // The channel for which the message belongs
          var channelGroup = m.subscription; // The channel group or wildcard subscription match (if exists)
          var pubTT = m.timetoken; // Publish timetoken
          var msg = m.message; // The Payload
          longitude = msg.longitude;
          latitude = msg.latitude;
          var publisher = m.publisher; //The Publisher
    var myLatlng = new google.maps.LatLng(latitude, longitude);
    var marker = new google.maps.Marker({
        position: myLatlng,
        title:"PubNub GPS"
    });
    // To add the marker to the map, call setMap();
    map.setCenter(marker.position);
    smoothZoom(map, 14, map.getZoom());
    marker.setMap(map);
      },
      presence: function(p) {
          // handle presence
          var action = p.action; // Can be join, leave, state-change or timeout
          var channelName = p.channel; // The channel for which the message belongs
          var occupancy = p.occupancy; // No. of users connected with the channel
          var state = p.state; // User State
          var channelGroup = p.subscription; //  The channel group or wildcard subscription match (if exists)
          var publishTime = p.timestamp; // Publish timetoken
          var timetoken = p.timetoken;  // Current timetoken
          var uuid = p.uuid; // UUIDs of users who are connected with the channel
      },
      status: function(s) {
          var affectedChannelGroups = s.affectedChannelGroups;
          var affectedChannels = s.affectedChannels;
          var category = s.category;
          var operation = s.operation;
      }
  });
  pubnub.subscribe({
      channels: ['ch1'],
  });
      var map;
      function initMap() {
        map = new google.maps.Map(document.getElementById('map'), {
          center: {lat: latitude, lng: longitude},
          zoom: 8
        });
      }
    </script>
    <script src="https://maps.googleapis.com/maps/api/js?key=AIzaSyBLuWQHjBa9SMVVDyyqxqTpR2ZwnxwcbGE&callback=initMap"
    async defer></script>
  </body>
</html>
```

This part of the code is responsible for rendering our map on the HTML page.

```
<style>
  /* Always set the map height explicitly to define the size of the div
   * element that contains the map. */
  #map {
    height: 100%;
  }
  /* Optional: Makes the sample page fill the window. */
  html, body {
    height: 100%;
    margin: 0;
    padding: 0;
  }
</style>
```

Just a little below it, we enter a div id tag to tell where we want the map to render:

  //div tag for map[] id
     <div id="map"></div>

Here we simply import the PubNub JS SDK to enable PubNub data streaming for our GPS data:

```
<script src="https://cdn.pubnub.com/sdk/javascript/pubnub.4.23.0.js"></script>
```

We must also import the Google Maps API with this script tag:

```
<script src="https://maps.googleapis.com/maps/api/js?key=YOURAPIKEY&callback=initMap"async defer></script>
```

NOTE: The rest of the code is encapsulated within one script tag, so don’t be alarmed if we jump around in explaining this final part of the code.

In order to stream our data, instantiate a PubNub instance:

```
var pubnub = new PubNub({
    subscribeKey: "YOUR SUBSCRIBE KEY",
    ssl: true
  });
```

Then we instantiate a PubNub listener with the following code.

```
pubnub.addListener({
      message: function(m) {
          // handle message
          var channelName = m.channel; // The channel for which the message belongs
          var channelGroup = m.subscription; // The channel group or wildcard subscription match (if exists)
          var pubTT = m.timetoken; // Publish timetoken
          var publisher = m.publisher; //The Publisher
          
          var msg = m.message; // The Payload
          //extract and save the longitude and latitude data from your incomming PubNub message
          longitude = msg.longitude;
          latitude = msg.latitude;
          
        //Create a new Google Maps instance with updated GPS coordinates
      var myLatlng = new google.maps.LatLng(latitude, longitude);
      //Create a marker instance with the coordinates
      var marker = new google.maps.Marker({
          position: myLatlng,
          title:"PubNub GPS"
      });
      
      //center the map with the maker position
      map.setCenter(marker.position);
      //Optional: create a zooming annimation when the gps changes coordinates
      smoothZoom(map, 14, map.getZoom());
      // To add the marker to the map, call setMap();
      marker.setMap(map);
      },
      presence: function(p) {
          // handle presence
          var action = p.action; // Can be join, leave, state-change or timeout
          var channelName = p.channel; // The channel for which the message belongs
          var occupancy = p.occupancy; // No. of users connected with the channel
          var state = p.state; // User State
          var channelGroup = p.subscription; //  The channel group or wildcard subscription match (if exists)
          var publishTime = p.timestamp; // Publish timetoken
          var timetoken = p.timetoken;  // Current timetoken
          var uuid = p.uuid; // UUIDs of users who are connected with the channel
      },
      status: function(s) {
          var affectedChannelGroups = s.affectedChannelGroups;
          var affectedChannels = s.affectedChannels;
          var category = s.category;
          var operation = s.operation;
      }
  });
```

In order to avoid syntax errors, place a subscriber instance right below the listener.

```
pubnub.subscribe({
      channels: ['YOUR CHANNEL NAME'],
  });
```

As you can see, we open up incoming messages with the following line of code.

```
var msg = m.message; // The Payload
```

And then extract the variables we desire based on the sent JSON.

```
longitude = msg.longitude;
latitude = msg.latitude;
```

We then format the data variables in accordance to a Google Maps object.

```
var myLatlng = new google.maps.LatLng(latitude, longitude);
```
To set a Google marker on our GPS coordinates we create a Google Maps marker object.

```
var marker = new google.maps.Marker({
          position: myLatlng,
          title:"Title of Marker"
      });
```

Then add the marker to your Google Maps object by calling setMap().

```
marker.setMap(map);
```

Of course, it would be nice to center our map on the marker so we can actually see it so we center it on the markers position.

```
map.setCenter(marker.position);
```

This is optional, but if you want to add a smooth zooming animation every time you locate a marker, call a smoothZoom function like so.

```
smoothZoom(map, 14, map.getZoom());
```

And implement the smoothZoom function somewhere.

```
function smoothZoom (map, max, cnt) {
      if (cnt >= max) {
          return;
      }
      else {
          z = google.maps.event.addListener(map, 'zoom_changed', function(event){
              google.maps.event.removeListener(z);
              smoothZoom(map, max, cnt + 1);
          });
          setTimeout(function(){map.setZoom(cnt)}, 80); // 80ms is what I found to work well on my system -- it might not work well on all systems
      }
  } 
```

Lastly we’ll need to initialize the map so we write:

```
var map;
     function initMap() {
       map = new google.maps.Map(document.getElementById('map'), {
         center: {lat: latitude, lng: longitude},
         zoom: 8
       });
     }
```
And set the initial values of your latitude and longitude variables to wherever you want.

```
var longitude = 30.5;
var latitude = 50.5;
```

And that’s it! 

