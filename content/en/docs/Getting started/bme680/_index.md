---
title: "Working with BME680 Sensors"
linkTitle: "Working with BME680 Sensors"
weight: 1200
description: >-
     Getting Started with BME680 Sensors
---

![image](https://collabnix.com/wp-content/uploads/2021/07/image-18.png)

## Hardware requirements:

- Jetson Nano: 2GB Model ($59)
- A 5V 4Amp charger
- 128GB SD card
- BME680 sensors

## Software requirements:

- Jetson SD card image from NVIDIA 
- Etcher software installed on your system


You can run RedisTimeSeries directly over an IoT Edge device. Follow the below steps to build RedisTimeSeries Docker Image over Jetson Nano:

## Verifying Docker version

SSH to 70.167.220.160 and install Docker

```
pico@pico1:~$ docker version
Client: Docker Engine - Community
 Version:           20.10.3
 API version:       1.41
 Go version:        go1.13.15
 Git commit:        48d30b5
 Built:             Fri Jan 29 14:33:34 2021
 OS/Arch:           linux/arm64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.6
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       8728dd2
  Built:            Fri Apr  9 22:43:42 2021
  OS/Arch:          linux/arm64
  Experimental:     false
 containerd:
  Version:          1.4.3
  GitCommit:        269548fa27e0089a8b8278fc4fc781d7f65a939b
 nvidia:
  Version:          1.0.0-rc92
  GitCommit:        ff819c7e9184c13b7c2607fe6c30ae19403a7aff
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

## Verifying if Sensor is detected
 
 ```
 i2cdetect -r -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- --
```

## Building Docker Image for RedisTimeSeries for Jetson Nano

```
git clone --recursive https://github.com/RedisTimeSeries/RedisTimeSeries.git
cd RedisTimeSeries.git
docker build -t ajeetraina/redistimeseries-jetson . -f Dockerfile.jetson.edge
```

## Running RedisTimeSeries

```
docker run -dit -p 6379:6379 ajeetraina/redistimeseries-jetson
```

## Verifying if RedisTimeSeries Module is loaded

```
redis-cli
127.0.0.1:6379> info modules
# Modules
module:name=timeseries,ver=999999,api=1,filters=0,usedby=[],using=[],options=[]
127.0.0.1:6379>
```

## Clone the repository

```
$ git clone https://github.com/redis-developer/redis-datasets
$ cd redis-datasets/redistimeseries/realtime-sensor-jetson
```

## Running Sensorload Script

```
sudo python3 sensorloader2.py --host localhost --port 6379
```

## Running Grafana on Jetson Nano

```
docker run -d -e "GF_INSTALL_PLUGINS=redis-app" -p 3000:3000 grafana/grafana
```

There you go..

Point your browser to https://<IP_ADDRESS>:3000. Use “admin” as username and password to log in to the Grafana dashboard.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-7-1024x982.png)

Click the Data Sources option on the left side of the Grafana dashboard to add a data source.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-8-1024x513.png)


Under the Add data source option, search for Redis and the Redis data source will appear as shown below:

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-9-1024x648.png)
![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-10-1024x489.png)

Supply the name, Redis Enterprise Cloud database endpoint, and password, then click Save & Test.

Click Dashboards to import Redis and Redis Streaming. Click Import for both these options.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-12-1024x425.png)

Click on Redis to see a fancy Grafana dashboard that shows the Redis database information:

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-13-1024x253.png)
![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-14-1024x475.png)

Finally, let’s create a sensor dashboard that shows temperature, pressure, and humidity. To start with temperature,  first click on + on the left navigation window. Under Create option, Select Dashboard and click on the Add new panel button.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-15-1024x445.png)

A new window will open showing the Query section. Select SensorT from the drop-down menu, choose RedisTimeSeries as type, TS.GET as command and ts”temperature as key.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-16-1024x461.png)

Choose TS.GET as a command.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-17-1024x404.png)

Type ts”temperature as the key.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-18-1024x371.png)

Click Run followed by Save, as shown below:

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-19-1024x415.png)

Now you can save the dashboard by your preferred name:

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-20.png)

Click Save. This will open up a sensor dashboard. You can click on Panel Title and select Edit.


![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-21-1024x542.png)


Type Temperature and choose Gauge under Visualization.

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-22-494x1024.png)

Click Apply and you should be able to see the temperature dashboard as shown here:

![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-23-1024x514.png)

Follow the same process for pressure (ts:pressure) and humidity (ts:humidity), and add them to the dashboard. You should be able to see the complete dashboard readings for temperature, humidity, and pressure. Looks amazing. Isn’t it?


![image](https://redislabs.com/wp-content/uploads/2021/02/ajeet-iot-24-1024x902.png)
