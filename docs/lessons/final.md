# Final Project

## Overview

I want to create a 3x2 solenoid array that can display braille characters by pushing solenoids up and down to create dots. This solenoid array will be connected to a Raspberry Pi, which in turn will be connected to an ESP32CAM. The camera will take a picture of a page of text, then perform OCR (optical character recognition) to extract a string of text from the image. That string of text will be converted to braille, which will be displayed on the solenoid array by flashing each character for 1 second at a time. This device will essentially allow for live-time conversion of any text into braille, which I hope will increase accessibility to books and the like.

## Brainstorming Process

### Initial Thoughts

My idea was to design a text to braille converter, which a blind person could use by moving the device over a page of text to convert it into braille. The braille translation of the English text would then be represented via a series of up/down pins which the user could use to interpret the information. The device was to be a rectangular box that would use an internal camera to interpret and OCR text, which could then be translated into braille and displayed via a series of servo motors pushing up metal rods on the top of the box. The pins would be in groups of six, each group representing a single braille character.

However, I talked to <a href="https://fabacademy.org/2023/labs/charlotte/students/stuart-christhilf/">**Stuart Christhilf**</a> who had thought of a similar mechanism for his initial final project. He originally planned to create a dynamic clock to display the time using blocks of wood that acould be pushed out or pulled back via servos. However, when building his project, he realized that fitting so many servos into such a small space was completely unfeasible and warned me from doing the same. My initial design is shown in the following image:

<center>
<img src="../../pics/week1/initialDesign.jpg" alt="Initial design" width="450"/>
</center>
<br>

I then decided to use electromagnets for my pins, instead of a servo. The pins themselves would be a small magnetic rod sitting on top of an electromagnet. The small electromagnet could be powered on and off via a microcontroller. When the electromagnet was off, the pin would simply rest on top of the electromagnet, and the pin would be flush against the top of the board, forming the down position of the pin. If the pin needed to pop up, the microcontroller would power the electromagnet which would then emit a repelling magnetic charge. That magnetic force would then repel the pin slightly upwards, forming the up position of the pin. To represent a braille character, the microcontroller would push the specific pins into the up position that together would form the 6-dot pattern of the character.

I also decided to move the camera out of the box. That would allow for more simple wiring and internal organization of the box, and allow the operator to more easily use the device. Moving the camera out means that the user would only need to move a small camera container across the page of text, instead of dragging the entire device. Here is my modified design:

<center>
<img src="../../pics/week1/modifiedDesign.jpg" alt="Modified design" width="700"/>
</center>
<br>

### Significant Changes

Although a large part of my project remains the same, I've changed some aspects of my project. Namely, I've decided to use a Raspberry Pi as a central controller and connect it to 5 separate ATTiny412 chips, which will each be responsible for controlling 6 electromagnets to represent 1 braille character. Each ATTiny412 and 6 electromagnet setup will be on its own PCB, and receive data from the controlling Raspberry Pi. Additionally, I decided to create an elevated case for the ESP32 camera so that the image would have a better angle and thus an easier time being processed for OCR, and so that more light could come into the camera lens from the unobstructed sides. Lastly, I decided I wanted to wirelessly transmit data from the ESP32 camera to the Raspberry Pi for processing. I worked with both serial communication and WiFi connectivity in previous weeks so I hope to sum it all together and wirelessly transmit data between these two controllers.

Here is an updated system diagram which maps out all the parts of my project.

<center>
<img src="../../pics/final/midterm/systemDiagram.jpg" width="700"/>
</center>

### Feasibility

However, after doing research, I realized that having 30 solenoids would be unfeasible. Instead, I decided to scale my project down to just having 6 solenoids, as this would still accomplish the mission of displaying braille for a reader. I would then flash each braille character for 1 second on the 6 solenoid array. This change allows me to worry less about power budget and ensures that I have a ready final project on my presentation date.

## Bill of Materials

<div style="text-align: center"> 
<iframe src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQlIJdCFYQU6-XJm1FrXhk5twaGxpRf5jiNvo1Z9Wf0MkVefTB23N4_w5QmfgFJcqXeWUzttINugkhU/pubhtml?widget=true&amp;chrome=false&amp;headers=false" frameborder="0" width="250%" height="300" scrolling="no"></iframe>
</div>


## Creation

### Braille Box CAD

#### Initial Design

I decided to first model my design in Fusion360, as I had prior experience working with Fusion and was pretty comfortable using it. When I started out with Autodesk Fusion, Kevin Kennedy's <a href="https://www.youtube.com/playlist?list=PLrZ2zKOtC_-C4rWfapgngoe9o2-ng8ZBr">**Fusion tutorials**</a> were a massive help.

I first started off with a rectangular prism to act as the main body of the design.

<center>
<img src="../../pics/week2/rectangularPrism.jpg" alt="Rectangular Prism Creation" width="700"/>
</center>

Next, I filleted the box to round out the edges.

<center>
<img src="../../pics/week2/fillet.jpg" alt="Fillet Creation" width="700"/>
</center>

I then created a sketch on the top of the box, where I created six circles. These 6 circles represent the holes where I will put metal pins into that can pop up and down depending on what needs to be represented.

<center>
<img src="../../pics/week2/holeSketch.jpg" alt="Sketching the circles" width="700"/>
</center>

I extruded the circles downward as holes. This creates the actual space where the pins will be placed.

<center>
<img src="../../pics/week2/holeExtrude.jpg" alt="Extruding the holes" width="500"/>
</center>

Finally, I used the pattern feature to repeat the sketch and extrusion across the top of the box. This created a total of 5 evenly spaced sets of 6 pins. With each set of 6 pins representing a single braille character, one iteration of pin setups can represent five letters.

<center>
<img src="../../pics/week2/pattern.jpg" alt="Patterning the holes" width="700"/>
</center>

#### Improved Design

As I had made my initial design early on in Fab Academy, it did not reflect the changes I had made to my final project, most notably scaling down the amount of solenoids from 5 arrays of 3x2 to one 3x2 array. Additionally, when I made the original design, I didn't think much about how I would power the solenoid array and thus didn't include any spots for batteries. I also wanted to make the holes for the solenoids on a separate press-fit cover on top of the main box. Finally, the original design doesn't include any internal parts to hold the solenoids in place.

For my new design, I want to make the following key changes:

 - 3x2 solenoid array
 - Internal beams to support solenoids
 - Battery pack holders
 - Press-fit cover

Additionally, I want the box to look as nice as possible and ideally have all wiring contained within it.

I first started out by creating the shell of the box. I hollowed out the innards because I want my electronics to be inside. I will end up adhering my PCB to the side of the box and having my MOSFET breakouts on the bottom of the hollowed inside.

<center>
<img src="../../pics/final/braillebox/bbshell.jpg" width="700"/>
</center>

I then started working on the top cover. I started out with creating a sketch where all my holes would be, and a offset on the edges to match the shell. I then extruded the sketch to create the cover with holes that the solenoids will fill.

<center>
    <table>
        <tr>
            <td><img src="../../pics/final/braillebox/bbholes1.jpg" width="500"/></td>
            <td><img src="../../pics/final/braillebox/bbholes2.jpg" width="700"/></td>
        </tr>
    </table>
</center>

Based on my previous sketch offset for the edges of the shell box, I created quarter-circles and extruded them to form the press-fit lid.

<center>
    <table>
        <tr>
            <td><img src="../../pics/final/braillebox/bbpressfit1.jpg" width="350"/></td>
            <td><img src="../../pics/final/braillebox/bbpressfit2.jpg" width="700"/></td>
        </tr>
    </table>
</center>

Next, I designed the beams that hold the solenoids in place. I started by creating a sketch on the bottom of the shell box and extruded that to my desired height. I then created a sketch on the extruded rectangular prism to remove the bottom part of it and form it into a beam-like shape.

<center>
    <table>
        <tr height="150%">
            <td><img src="../../pics/final/braillebox/bbbeams1.jpg" width="500"/></td>
            <td><img src="../../pics/final/braillebox/bbbeams2.jpg" width="700"/></td>
            <td><img src="../../pics/final/braillebox/bbbeams3.jpg" width="700"/></td>
        </tr>
    </table>
</center>

I then started work on the battery holders. I created the bottom of the battery holder then extruded out the sides.

<center>
    <table>
        <tr>
            <td><img src="../../pics/final/braillebox/bbbatterybottom.jpg" width="600"/></td>
            <td><img src="../../pics/final/braillebox/bbbatteryshelled.jpg" width="500"/></td>
        </tr>
    </table>
</center>

Next, I created the dividers to firmly hold each battery pack in place. Each divided section has the same length and width dimensions as the actual battery pack that I will use, plus a little for tolerance.

<center>
    <table>
        <tr>
            <td><img src="../../pics/final/braillebox/bbbatterydividersketch.jpg" width="500"/></td>
            <td><img src="../../pics/final/braillebox/bbbatterydivided.jpg" width="500"/></td>
        </tr>
    </table>
</center>

I then added a small hole on the side for the power, ground, and TX/RX cables for the ATTiny1614.

<center>
<img src="../../pics/final/braillebox/bbwirehole.jpg" width="500"/>
</center>

Next, I extruded a small hole as a slot for the wires on the external battery packs to route into the main shell box, where it will be connected to the MOSFETs controlling the solenoids.

<center>
<img src="../../pics/final/braillebox/bbbatterywireslot.jpg" width="500"/>
</center>

Finally, I added fillets. Here is the final box design.

<center>
<img src="../../pics/final/braillebox/bbfillets.jpg" width="500"/>
</center>

### Raspberry Pi Box CAD

I first started off with a shelled box.

<center>
<img src="../../pics/final/rpibox/rpbshelledbox.jpg" width="500"/>
</center>

I then added a lid, with a hole the same size as my 5 inch touchscreen where I would attach the screen.

<center>
<img src="../../pics/final/rpibox/rpblid.jpg" width="500"/>
</center>

Next, I added screw holes on the lid. These screw holes allowed me to secure the screen to the 3D printed lid. Unfortunately, my screw holes actually ended up being a little small so I had to enlarge them after the print with the help of a soldering iron.

<center>
<img src="../../pics/final/rpibox/rpblidscrewholes.jpg" width="500"/>
</center>

I added legs to the lid to allow it to press-fit into the base.

<center>
<img src="../../pics/final/rpibox/rpblegs.jpg" width="500"/>
</center>

Then, I added holes for wires on the front and side of the Pi case.

<center>
    <table>
        <tr>
            <td><img src="../../pics/final/rpibox/rpbwirehole.jpg" width="500"/></td>
            <td><img src="../../pics/final/rpibox/rpbwirehole2.jpg" width="500"/></td>
        </tr>
    </table>
</center>

Finally, I added fillets all around.

<center>
<img src="../../pics/final/rpibox/rpbfillets.jpg" width="500"/>
</center>

After printing that initial iteration, the screen fit and there was enough space inside to fit the Raspberry Pi. However, some of my cables didn't fit as they had long "necks" that had to remain straight. As such, I would have to significantly bend the HDMI and USB cords for the Raspberry Pi.

<center>
<img src="../../pics/final/rpibox/rpbusbneck.jpg" width="500"/>
</center>

As such, I adjusted the length of the box to give space for the USB cable necks. I also slightly decreased the height of the holes for the USB cables as they were larger than necessary and somewhat an eyesore.

<center>
<img src="../../pics/final/rpibox/rpbsizeAdjustments.jpg" width="500"/>
</center>

### PCB Production

to be documented

### ESP32CAM Wireless Transmission

WebSocket connections are initiated through HTTP protocol, using an upgrade request from HTTP to WebSocket. This begins with a client sending a standard HTTP request that includes an "Upgrade: websocket" header and a "Connection: Upgrade" header to the server. The server then responds with an HTTP 101 status code, indicating that the protocol will change, thus establishing the WebSocket connection.

WebSocket, uses IP addresses to facilitate the initial connection before upgrading to the WebSocket protocol. Once the WebSocket connection is established, the IP addresses are used to maintain the connection over which data frames can be reliably transmitted back and forth.

Once the WebSocket connection is established, data is transmitted in framed messages through backend data transmission ports, where each frame consists of an opcode to indicate the type of data being transmitted (e.g., text, binary, continuation frame, or control frames like close, ping, or pong). This structure allows the WebSocket protocol to be extremely versatile and efficient in handling different types of data seamlessly. The frames are small and allow for very efficient data transmission.

The following program is uploaded onto the ESP32 CAM Board through Arduino IDE. This program requires a lot of libraries and dependencies, which I individually downloaded from Github and can be found on my Week 13 documentation. This program is based off of the CameraWebServer example program from ESP32.

```cpp
#include "esp_camera.h"
#include "WiFi.h"
#include "WebSocketsServer.h"

#define CAMERA_MODEL_AI_THINKER // Has PSRAM
#include "camera_pins.h"

const char* ssid = "REDACTED";
const char* password = "REDACTED";

WebSocketsServer webSocket = WebSocketsServer(81);

void startCameraServer();
void setupLedFlash(int pin);
void onWebSocketEvent(uint8_t client_num, WStype_t type, uint8_t *payload, size_t length);

void setup() {
  pinMode(2, OUTPUT);
  Serial.begin(9600);
  while (!Serial); // Wait for the serial connection to initialize
  Serial.setDebugOutput(true);
  Serial.println();

  camera_config_t config;
  config.ledc_channel = LEDC_CHANNEL_0;
  config.ledc_timer = LEDC_TIMER_0;
  config.pin_d0 = Y2_GPIO_NUM;
  config.pin_d1 = Y3_GPIO_NUM;
  config.pin_d2 = Y4_GPIO_NUM;
  config.pin_d3 = Y5_GPIO_NUM;
  config.pin_d4 = Y6_GPIO_NUM;
  config.pin_d5 = Y7_GPIO_NUM;
  config.pin_d6 = Y8_GPIO_NUM;
  config.pin_d7 = Y9_GPIO_NUM;
  config.pin_xclk = XCLK_GPIO_NUM;
  config.pin_pclk = PCLK_GPIO_NUM;
  config.pin_vsync = VSYNC_GPIO_NUM;
  config.pin_href = HREF_GPIO_NUM;
  config.pin_sscb_sda = SIOD_GPIO_NUM;
  config.pin_sscb_scl = SIOC_GPIO_NUM;
  config.pin_pwdn = PWDN_GPIO_NUM;
  config.pin_reset = RESET_GPIO_NUM;
  config.xclk_freq_hz = 20000000;
  config.frame_size = FRAMESIZE_UXGA;
  config.pixel_format = PIXFORMAT_JPEG; 
  config.grab_mode = CAMERA_GRAB_WHEN_EMPTY;
  config.fb_location = CAMERA_FB_IN_PSRAM;
  config.jpeg_quality = 12;
  config.fb_count = 1;

  if (psramFound()) {
    config.jpeg_quality = 10;
    config.fb_count = 2;
    config.grab_mode = CAMERA_GRAB_LATEST;
  }

  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }

  sensor_t *s = esp_camera_sensor_get();
  s->set_vflip(s, 1); // Flip it back
  s->set_brightness(s, 1); // Up the brightness just a bit
  s->set_saturation(s, -2); // Lower the saturation

#if defined(LED_GPIO_NUM)
  setupLedFlash(LED_GPIO_NUM);
#endif

  WiFi.begin(ssid, password);
  WiFi.setSleep(false);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  webSocket.begin();
  webSocket.onEvent(onWebSocketEvent);
  startCameraServer();

  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
}

void loop() {
  webSocket.loop();
}

void onWebSocketEvent(uint8_t client_num, WStype_t type, uint8_t *payload, size_t length) {
  switch (type) {
    case WStype_DISCONNECTED:
      Serial.printf("[%u] Disconnected!\n", client_num);
      break;
    case WStype_CONNECTED:
      {
        IPAddress ip = webSocket.remoteIP(client_num);
        Serial.printf("[%u] Connection from ", client_num);
        Serial.println(ip.toString());
      }
      break;
    case WStype_TEXT:
      if (strcmp((char *)payload, "capture") == 0) {
        camera_fb_t *fb = esp_camera_fb_get();
        if (!fb) {
          Serial.println("Camera capture failed");
        } else {
          webSocket.sendBIN(client_num, fb->buf, fb->len);
          esp_camera_fb_return(fb);
        }
      }
      break;
    case WStype_BIN:
      Serial.printf("[%u] Get binary length: %u\n", client_num, length);
      break;
  }
}

void setupLedFlash(int pin) {
  pinMode(pin, OUTPUT);
  digitalWrite(pin, LOW);
}
```

This program connects the ESP32CAM to a local WiFi network. It then sets up and initializes the camera, and sets up the local IP connection. It then continuously waits for a web socket connection. When a connection is created, it prints the IP address of the connecting device. If the device sends an input of "capture", the camera will take a picture and send it via the network web socket connection to the connecting Raspberry Pi.

## Camera Feed OCR

//todo: swap pytesseract model out for gpt-4o

During <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week13/networking/">**Networking Week**</a>, I had already setup infrastructure to wirelessly transmit a command to capture an image from a Raspberry Pi to the ESP32CAM, along with sending the image data back over the network and saving it. I had created a WebSocket server to accept commands and then send the image data over HTTP back to the Raspberry Pi.

This week, however, I realized that I could fetch the image without needing a WebSocket handler by connecting to the ESP32CAM's capture image handler directly. The capture handler from the default CameraWebServer example project sets up a port that allows a direct download to what is currently on the camera feed.

```cpp
static esp_err_t capture_handler(httpd_req_t *req)
{
    camera_fb_t *fb = NULL;
    esp_err_t res = ESP_OK;
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
    int64_t fr_start = esp_timer_get_time();
#endif

#if CONFIG_LED_ILLUMINATOR_ENABLED
    enable_led(true);
    vTaskDelay(150 / portTICK_PERIOD_MS); // The LED needs to be turned on ~150ms before the call to esp_camera_fb_get()
    fb = esp_camera_fb_get();             // or it won't be visible in the frame. A better way to do this is needed.
    enable_led(false);
#else
    fb = esp_camera_fb_get();
#endif

    if (!fb)
    {
        log_e("Camera capture failed");
        httpd_resp_send_500(req);
        return ESP_FAIL;
    }

    httpd_resp_set_type(req, "image/jpeg");
    httpd_resp_set_hdr(req, "Content-Disposition", "inline; filename=capture.jpg");
    httpd_resp_set_hdr(req, "Access-Control-Allow-Origin", "*");

    char ts[32];
    snprintf(ts, 32, "%lld.%06ld", fb->timestamp.tv_sec, fb->timestamp.tv_usec);
    httpd_resp_set_hdr(req, "X-Timestamp", (const char *)ts);

#if CONFIG_ESP_FACE_DETECT_ENABLED
    size_t out_len, out_width, out_height;
    uint8_t *out_buf;
    bool s;
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
    bool detected = false;
#endif
    int face_id = 0;
    if (!detection_enabled || fb->width > 400)
    {
#endif
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
        size_t fb_len = 0;
#endif
        if (fb->format == PIXFORMAT_JPEG)
        {
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
            fb_len = fb->len;
#endif
            res = httpd_resp_send(req, (const char *)fb->buf, fb->len);
        }
        else
        {
            jpg_chunking_t jchunk = {req, 0};
            res = frame2jpg_cb(fb, 80, jpg_encode_stream, &jchunk) ? ESP_OK : ESP_FAIL;
            httpd_resp_send_chunk(req, NULL, 0);
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
            fb_len = jchunk.len;
#endif
        }
        esp_camera_fb_return(fb);
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
        int64_t fr_end = esp_timer_get_time();
#endif
        log_i("JPG: %uB %ums", (uint32_t)(fb_len), (uint32_t)((fr_end - fr_start) / 1000));
        return res;
#if CONFIG_ESP_FACE_DETECT_ENABLED
    }

    jpg_chunking_t jchunk = {req, 0};

    if (fb->format == PIXFORMAT_RGB565
#if CONFIG_ESP_FACE_RECOGNITION_ENABLED
     && !recognition_enabled
#endif
     ){
#if TWO_STAGE
        HumanFaceDetectMSR01 s1(0.1F, 0.5F, 10, 0.2F);
        HumanFaceDetectMNP01 s2(0.5F, 0.3F, 5);
        std::list<dl::detect::result_t> &candidates = s1.infer((uint16_t *)fb->buf, {(int)fb->height, (int)fb->width, 3});
        std::list<dl::detect::result_t> &results = s2.infer((uint16_t *)fb->buf, {(int)fb->height, (int)fb->width, 3}, candidates);
#else
        HumanFaceDetectMSR01 s1(0.3F, 0.5F, 10, 0.2F);
        std::list<dl::detect::result_t> &results = s1.infer((uint16_t *)fb->buf, {(int)fb->height, (int)fb->width, 3});
#endif
        if (results.size() > 0) {
            fb_data_t rfb;
            rfb.width = fb->width;
            rfb.height = fb->height;
            rfb.data = fb->buf;
            rfb.bytes_per_pixel = 2;
            rfb.format = FB_RGB565;
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
            detected = true;
#endif
            draw_face_boxes(&rfb, &results, face_id);
        }
        s = fmt2jpg_cb(fb->buf, fb->len, fb->width, fb->height, PIXFORMAT_RGB565, 90, jpg_encode_stream, &jchunk);
        esp_camera_fb_return(fb);
    } else
    {
        out_len = fb->width * fb->height * 3;
        out_width = fb->width;
        out_height = fb->height;
        out_buf = (uint8_t*)malloc(out_len);
        if (!out_buf) {
            log_e("out_buf malloc failed");
            httpd_resp_send_500(req);
            return ESP_FAIL;
        }
        s = fmt2rgb888(fb->buf, fb->len, fb->format, out_buf);
        esp_camera_fb_return(fb);
        if (!s) {
            free(out_buf);
            log_e("To rgb888 failed");
            httpd_resp_send_500(req);
            return ESP_FAIL;
        }

        fb_data_t rfb;
        rfb.width = out_width;
        rfb.height = out_height;
        rfb.data = out_buf;
        rfb.bytes_per_pixel = 3;
        rfb.format = FB_BGR888;

#if TWO_STAGE
        HumanFaceDetectMSR01 s1(0.1F, 0.5F, 10, 0.2F);
        HumanFaceDetectMNP01 s2(0.5F, 0.3F, 5);
        std::list<dl::detect::result_t> &candidates = s1.infer((uint8_t *)out_buf, {(int)out_height, (int)out_width, 3});
        std::list<dl::detect::result_t> &results = s2.infer((uint8_t *)out_buf, {(int)out_height, (int)out_width, 3}, candidates);
#else
        HumanFaceDetectMSR01 s1(0.3F, 0.5F, 10, 0.2F);
        std::list<dl::detect::result_t> &results = s1.infer((uint8_t *)out_buf, {(int)out_height, (int)out_width, 3});
#endif

        if (results.size() > 0) {
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
            detected = true;
#endif
#if CONFIG_ESP_FACE_RECOGNITION_ENABLED
            if (recognition_enabled) {
                face_id = run_face_recognition(&rfb, &results);
            }
#endif
            draw_face_boxes(&rfb, &results, face_id);
        }

        s = fmt2jpg_cb(out_buf, out_len, out_width, out_height, PIXFORMAT_RGB888, 90, jpg_encode_stream, &jchunk);
        free(out_buf);
    }

    if (!s) {
        log_e("JPEG compression failed");
        httpd_resp_send_500(req);
        return ESP_FAIL;
    }
#if ARDUHAL_LOG_LEVEL >= ARDUHAL_LOG_LEVEL_INFO
    int64_t fr_end = esp_timer_get_time();
#endif
    log_i("FACE: %uB %ums %s%d", (uint32_t)(jchunk.len), (uint32_t)((fr_end - fr_start) / 1000), detected ? "DETECTED " : "", face_id);
    return res;
#endif
}
```

I then set up the Raspberry Pi to receive an image from the ESP32CAM and perform OCR upon it.

First, I created a directory to store this project.

```
cd Desktop
mkdir ocr
```

Upon entering the new directory, I need to create a virtual environment to install the libraries I will be using for OCR.

```cmd
python -m venv /virtual
```

However, running this command gave me an error. 

```
Error: [Errno13] Permission denied: '/virtual'
```

For some reason, this command didn't have the permissions to create a new virtual environment, which was strange considering that the project directory was not protected in any way. Regardless, I attached the sudo prefix and successfully created the virtual environment.

```
sudo python -m venv /virtual
```

I then entered the virtual environment by activating it.

```
source bin/activate
```

The bin/activate is a relative path and would activate the venv as long as I am in the "ocr" folder. However, the venv could also be activated by supplying the absolute path of "~/home/richard/Desktop/icr/bin/activate".

After activating the virtual environment, I can install all of my library dependencies.

```
sudo pip install pytesseract
sudo pip install opencv-python
```

I then created the actual program that the Raspberry Pi would run.

```py
import time
import cv2
import urllib.request
import numpy as np
import pytesseract

url = 'http://10.12.28.193/capture'

img_resp = urllib.request.urlopen(url)
imgnp = np.array(bytearray(img_resp.read()), dtype=np.uint8)
frame = cv2.imdecode(imgnp, -1)

text = pytesseract.image_to_string(frame, config='--psm 7')

print("Extracted Text:", text)
time.sleep(1)
```

This script has the Raspberry Pi connect to the /capture handler of the ESP32CAM interface, which directly returns a capture of the current feed. It then decodes the image and parses it into the pytesseract OCR function. The PSM value of 6 tells the OCR model to scan the image for a single text block and extract text from that. A full list of PSM value options can be found by running ```tesseract --help-psm``` in the terminal.

```
  0    Orientation and script detection (OSD) only.
  1    Automatic page segmentation with OSD.
  2    Automatic page segmentation, but no OSD, or OCR. (not implemented)
  3    Fully automatic page segmentation, but no OSD. (Default)
  4    Assume a single column of text of variable sizes.
  5    Assume a single uniform block of vertically aligned text.
  6    Assume a single uniform block of text.
  7    Treat the image as a single text line.
  8    Treat the image as a single word.
  9    Treat the image as a single word in a circle.
 10    Treat the image as a single character.
 11    Sparse text. Find as much text as possible in no particular order.
 12    Sparse text with OSD.
 13    Raw line. Treat the image as a single text line, bypassing hacks that are Tesseract-specific.
```

In my case, since I want the model to scan an image to find the line of text for "Hello World!", I will use psm-7.

Here is a photo of my Raspberry Pi setup.

<center>
<img src="../../pics/week15/setup.jpg" width="500"/>
</center>

The ESP32CAM is pointed towards a paper with the words "Hello World!". In the right side of the picture, the Raspberry Pi which is running the code is visible along with the display. Upon running the program on the Pi's terminal, the ESP32CAM takes a picture and transmits it to the Pi, which then uses tesseract to perform OCR on it and prints out the extracted text.

<center>
<video width="550" height="300" controls><source src="../../pics/week15/pi.mp4" type="video/mp4" /></video>
</center>



## Midterm Review

### Significant Changes

Although a large part of my project remains the same, I've learned a lot over the past 10 weeks and have changed some aspects of my project. Namely, I've decided to use a Raspberry Pi as a central controller and connect it to 5 separate ATTiny412 chips, which will each be responsible for controlling 6 electromagnets to represent 1 braille character. Each ATTiny412 and 6 electromagnet setup will be on its own PCB, and receive data from the controlling Raspberry Pi. Additionally, I decided to create an elevated case for the ESP32 camera so that the image would have a better angle and thus an easier time being processed for OCR, and so that more light could come into the camera lens from the unobstructed sides. Lastly, I decided I wanted to wirelessly transmit data from the ESP32 camera to the Raspberry Pi for processing. I worked with both serial communication and WiFi connectivity in previous weeks so I hope to sum it all together and wirelessly transmit data between these two controllers.

### System Diagram

Here is an updated system diagram which maps out all the parts of my project.

<center>
<img src="../../pics/final/midterm/systemDiagram.jpg" width="700"/>
</center>

### Tasks

I made a list of the tasks I need to complete in the upcoming weeks for my final project.

 - ☐ design and print a PCB for each set of electromagnets
 - ☐ test simultaneously controlling 6 different microcontrollers with RPi through different data output pins
 - ☐ develop Bluetooth communication between ESP32 and RPi
 - ☐ figure out how electromagnets work
 - ☐ research processing capabilities of ESP32 and decide whether to process images on the ESP32 or on the RPi
 - ☐ assemble the project
 - ☑ design a custom case
 - ☑ integrate with WiFi
 - ☑ test serial communication
 - ☑ find Python OCR library (pytesseract)
 - ☑ find Python library to convert text to braille (https://github.com/AaditT/braille - possibly used for OCR too?)

### Schedule for Completion

I created a Gantt chart to map out when I could finish tasks for my final.

<center>
<img src="../../pics/final/midterm/gantt.jpg" width="700"/>
</center>

As is shown in the chart, most of my remaining work can be during a week that corresponds with the task at hand. In the cases that don't, I plan to spend extra time completing those tasks during less intensive weeks.

## May 2024 Progress Check

 - ☐ design and print a PCB for each set of electromagnets <b>(waiting for actual solenoids to arrive)</b>
 - ☐ test simultaneously controlling 6 different microcontrollers with RPi through different data output pins
 - ☑ develop Bluetooth communication between ESP32 and RPi
 - ☐ figure out how electromagnets work
 - ☐ text to braille conversion
 - ☑ research processing capabilities of ESP32 and decide whether to process images on the ESP32 or on the RPi
 - ☐ assemble the project
 - ☑ design a custom case
 - ☑ integrate with WiFi
 - ☑ test serial communication
 - ☑ find Python OCR library (pytesseract)
 - ☑ find Python library to convert text to braille (https://github.com/AaditT/braille - possibly used for OCR too?)

Once my parts arrive, I need to assemble them onto a board and debug any potential issues that <i>definitely</i> will arise. I then need to assemble the system together and test out displaying multiple braille characters using multiple microcontrollers. In the meantime, I will be working on converting text into a 3x2 array representing braille dots.

