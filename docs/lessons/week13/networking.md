# Networking

I decided to use this week's assignment to further develop my final project. I thus decided to develop two programs. One program would be uploaded to the ESP32CAM, which listens and waits for a remote connection via its local IP address on a common WiFi network. The ESP32 will then take a picture when prompted by the remote connection then send the data of the image over Websocket HTTP to a receiving device. The second program is a python script running on a computer or a Raspberry Pi. This script initiates a Websocket connection to the IP of the ESP32CAM, and then receives the image, which is saved locally on the main device.

## ESP32CAM Board Program

The following program is uploaded onto the ESP32 CAM Board through Arduino IDE. This program requires a lot of libraries and dependencies, which I individually downloaded from Github. The entire project folder including the code and libraries can be downloaded under <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week13/week13Downloads/">**this week's file downloads**</a>. 

<pre><code class="language-cpp">