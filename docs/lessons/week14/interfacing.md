# Camera Interface

I wanted to align my work this week to help me on developing my final project. As such, I chose to create a web GUI to interact with the ESP32CAM. My documentation for setting up the ESP32CAM can be found on my documentation for <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week11/cameraFRT/">**Input Devices**</a> and <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week13/networking/">**Networking**</a>. 

To create a web-based locally hosted GUI to access the ESP32CAM, I need to do a few things:

 - Connect to the WiFi network
 - Receive an assigned IP from the router
 - At the IP address, locate an open port
 - Setup URI paths for backend control (e.g. buttons for taking picture) (/status gives status) - they are all serviced on the same port but differ by path
 - Setup URI path for the actual camera access. (IPADDRESS/capture is used for taking a picture)
 - Startup the server on the IP
 - Serve the graphical user interface HTML onto the IP
 - Upon receiving an input, the HTML will do something (what is this something?) and send the request along a URI path designed for handling that.

