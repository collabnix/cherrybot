# Install Pubnub

```
pip3 install pubnub
```

## cd into the examples directory containing the gps_simpletest.py file and install the PubNub Python SDK.

```
pip3 install pubnub
```

## Import PubNub packages.

```
import pubnub
from pubnub.pnconfiguration import PNConfiguration
from pubnub.pubnub import PubNub
from pubnub.callbacks import SubscribeCallback
from pubnub.enums import PNOperationType, PNStatusCategory
Configure a PubNub instance with your publish/subscribe keys.

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

