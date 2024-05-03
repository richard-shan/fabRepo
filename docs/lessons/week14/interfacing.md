# Camera Interface

I wanted to align my work this week to help me on developing my final project. As such, I chose to create a web GUI to interact with the ESP32CAM. My documentation for setting up the ESP32CAM can be found on my documentation for <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week11/cameraFRT/">**Input Devices**</a> and <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week13/networking/">**Networking**</a>. 

## Outline
To create a web-based locally hosted GUI to access the ESP32CAM, I need to do a few things:

 - Connect to the WiFi network
 - Receive an assigned IP from the router
 - Choose an open port
 - Setup URI paths for backend control (e.g. buttons for taking picture) (/status gives status) - they are all serviced on the same port but differ by path
 - Setup URI path for the actual camera access. (IPADDRESS/capture is used for taking a picture)
 - Startup the server on the IP
 - Start listening for incoming connections on the specified IP address and port
 - Serve the graphical user interface HTML onto the IP's root path
 - Upon receiving an input, the HTML will do something (what is this something?) and send the request along a URI path designed for handling that.

## WiFi Connection

First, the ESP32CAM connects to a WiFi network.

<pre><code class="language-cpp">const char* ssid = "REDACTED";
const char* password = "REDACTED";

  WiFi.begin(ssid, password);
  WiFi.setSleep(false);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
</code></pre>

This code connects the chip to the specified network and logs its success.

## Receive Assigned IP

The IP address assignment is handled automatically by the DHCP server on the network once the WiFi connection is established. This job is done by the server and without any action on the ESP32CAM's part, meaning that there is no code for IP assignment. However, the assigned IP address is then logged with:

<pre><code class="language-cpp">
Serial.print("Camera Ready! Use 'http://");
Serial.print(WiFi.localIP());
Serial.println("' to connect");
</code></pre>

## Choose Port

This code from app_httpd.cpp initializes the HTTP server configuration and starts the server on the default port.

<pre><code class="language-cpp">httpd_config_t config = HTTPD_DEFAULT_CONFIG();</code></pre>


