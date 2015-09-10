# Home Monitoring with Raspberry Pi and Node.js
 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/system.jpg?raw=true "Home Monitoring with Raspberry Pi and Node.js")
 
## Description  
 
The project is designed as a end to end solution for a DIY Home Monitoring & Intruder Alert system. Besides offering a live video stream on any device (web responsive client), it also actively monitors for movement with the help of a PIR sensor.   
 
If an Alarm is triggered, you get a SMS notification on your phone and the snapshots taken during the Alarm time span (customizable - default is 10 minutes) are uploaded via FTP to your server.  
 
Activation / Deactivation of the Alarm Mode can be done in 2 ways:  
1. from the Web Client user interface 
2. with a Button - for convenience reasons: it is faster than connecting from your phone / pc & toggling the Alert Mode checkbox 
    - you simply toggle the Alert mode with the press of a button  
    - there is a 10 seconds customizable delay which allows you to move out of the PIR sensor range 
    - a Led indicates the Alarm Mode enabled/disabled status 
 
In order to avoid false positives from the PIR motion sensor, extra checks were added - a detection counter & detection interval - in order for the Alarm to get triggered, the sensor needs to detect movement 3 times in 5 seconds (both values configurable in code). 
 
 
## Source Code 
 
The source code is open source (MIT License)
 
## Technology 
 
The project was developed using: 
- [Raspberry Pi](http://raspberrypi.org) - [raspbian](https://www.raspbian.org/), brick button & led, Pir sensor
- [Node.js](https://nodejs.org/en/) - for the main application 
- [Mjpg_streamer](http://sourceforge.net/projects/mjpg-streamer/) - to generate the video stream 
- Shell scripting - for easy application start (interactive & background) 
- Htms/Css/Javascript + [Bootstrap](http://getbootstrap.com/) - the web client  
 
## Project components 
 
### Hardware 

- Raspberry Pi 
  - I used *Model B Revision 2* with *Raspbian* - any model should be ok, just be careful with the Gpio configuration pin mappings, they can differ 
  - Generic USB webcam (compatible with Raspberry Pi & Raspbian) 
  - You can find a comprehensive list here http://elinux.org/RPi_USB_Webcams  
  - I used a very old 2MP one which seems to work out of the box with the generic drivers 
- Led & Button 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/button-led-brick.png?raw=true "Brick Button Led")
- PIR motion sensor 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/pir.jpg?raw=true "PIR sensor")
  - The one I used is available here https://www.sparkfun.com/products/13285  
  - It normally connects to Analog Input (ex. on Arduino); however you can use it with Digital as well if you connect a 10K resistor between VCC & Signal  
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/pir-10k-resistor.jpg?raw=true "PIR resistor") 
  - To make things easier you can purchase this sensor https://www.adafruit.com/products/189 and skip the soldering part (+ this one has configurable sensitivity built-in, so you might be able to skip the one implemented in the code)    
 
### Node application 
 
#### Dependencies 
- express: ^4.12.3 
- ftp: ^0.3.10 
- http-auth: ^2.2.8 
- ini: ^1.3.4 
- pi-gpio: 0.0.7 
- socket.io: ^1.3.5 
- twilio: ^2.3.0 
 
The dependencies you install with NPM: 
```
npm install module --save
```
 
#### Generic ```Application.js```
It is the basic application object, defined to be reusable in other projects 
Contains the basic server code, generic config file read/write operations, generic Init & Execute & Exit methods implementations 
 
#### Home Monitoring ```ApplicationHM.js``` 
- ```config.ini``` file 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/config.PNG?raw=true "Config")
- Authentication (digest http authentication) - defaults are **admin** & **password** :) 
  - You can change them from the ```htdigest``` file (nice helper tool here http://websistent.com/tools/htdigest-generator-tool/ ) 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/authentication-htdigest.PNG?raw=true "Http Digest Authentication")  
- Web Client application (with Websockets) 
  - Accessible from anywhere via [port forwarding](https://en.wikipedia.org/wiki/Port_forwarding)
  - Available also on mobile (responsive web client) 
- Monitoring - gets video from [Mjpg_streamer](http://sourceforge.net/projects/mjpg-streamer/) server and sends it to the connected app clients 
- [Mjpg_streamer](http://sourceforge.net/projects/mjpg-streamer/) was used as server, but if you prefer another tool like ffmpeg, you can easily replace it because of the loose integration via the ```start-webcam.sh``` script 

#### Alarm mode 
- Monitoring - via PIR sensor 
- Alarm - Sms notification (implemented with the help of [Twilio](https://www.twilio.com/sms)   text messaging API - very cool service, offers great Trial account for development 
- Alarm - Snapshots upload to server via Ftp 
 
### Web Client - responsive
 
The client application was designed to be accessible on all platforms (pc / tablet / mobile). 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/client.PNG?raw=true "Web Client")  

#### Video streaming quality settings 
By default the 480p at 25fps is enabled (initial settings are loaded from the ```config.ini``` file) 

My webcam is a low-end 5+ years old 2mp device, but for those of you with better webcams I also added 720p & 1080p 

Video resolutions & fps can be configured from the ```/static/js/script.js``` file 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/script-config.PNG?raw=true "Video Quality Configuration")  
 
#### Alert Mode 
- initial state is loaded from the ```config.ini``` file 
- You can enable/disable monitoring from checkbox button in the UI 
- The state of the Alert Mode is shown both in the UI (the checkbox) but also by the LED 
- The physical Button can be also used to toggle the Alert Mode 
- All state changes are sent to all connected clients 
- If an Alarm is triggered, the UI checkbox button background will be changed to Red 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/alarm.PNG?raw=true "Alarm")  

 
#### Connected Clients 
The dropdown shows a list of all connected clients (connection timestamp & IP) that are currently viewing the video stream  
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/clients-list.PNG?raw=true "Connected Clients List")  

 
### Shell Scripts 
 
```
start-app.sh
``` 
- You can start the application in 2 modes: 
  - Interactive (for dev / testing): ```./start-app.sh```
  - Background: ```./start-app.sh -background```
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/start-app.PNG?raw=true "Start App Script")  

  
```
start-webcam.sh  
```
- Used by the application to enable/disable video streaming when clients are connected or when an Alarm is triggered by the PIR sensor. 
![Alt text](https://github.com/orosandrei/Home-Monitoring-Raspberry-Pi-Node/raw/master/screenshots/start-webcam.PNG?raw=true "Start Webcam Script")  

---
 
**TO DO**
- Some code clean-up & optimizations  
- Port the application to Windows 10 Iot on Raspberry Pi 2 
- Support for uploading snapshots to cloud (OneDrive / Dropbox) when an Alarm is triggered 
 
**References** 
- Raspberry Pi https://www.raspberrypi.org/  
- Node - https://nodejs.org/en/  
- Mjpg_streamer http://sourceforge.net/projects/mjpg-streamer/  
- SMS Api - Twilio - https://www.twilio.com/sms  
- Bootstrap http://getbootstrap.com/ 
- App Webcam Icon - https://www.iconfinder.com/icons/71274/webcam_icon#size=128  
---
**Links**
- [Hackster.io project](https://www.hackster.io/andreioros) 
- twitter [@orosandrei](https://twitter.com/orosandrei)