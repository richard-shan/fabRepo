# ESP-32 Camera Facial Recognition

## Assignment

For this week, I was assigned to measure something using a sensor and take readings from it.

## Camera Board

We had ESP32 boards that were already available in the lab, however, those boards did not have hardware to support camera integration nor did they have a USB port. As such, I needed to buy a ESP-32 Camera Board specifically, instead of a general ESP32 board. I also needed a USB shield that would adapt the ESP32 Camera Board to a USB port that I could connect to my computer. The following image shows an ESP32 Camera Board with an OV2640 camera attached to it and is connected to a USB shield adapter.

<center>
<img src="../../../pics/week11/esp32camBoard.jpg" width="300"/>
</center>

The ESP32 board also comes with built-in WiFi connectivity which allows for remote access to the camera feed. This means that this camera can be deployed essentially off the shelf as a recording security camera, but for my purposes, I used this capability to be able to plug the board directly into a wall outlet and control the board from my computer which is connected to the same WiFi network.

## Programming

I first installed <a href="https://github.com/espressif/arduino-esp32">**Espressif's ESP32 board library**</a>  in Arduino IDE. I also added the board (```https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json```) to Arduino IDE's Additional Boards Manager URLs menu under ```File -> Preferences -> Additional boards managers URLs```. 

After this, I selected the board under ```Tools -> Board```.

<center>
<img src="../../../pics/week11/boardArduinoIDE.jpg" width="500"/>
</center>

After I connected to the board, I loaded the following program which would create a locally hosted web server that allows me to control the camera.

<pre><code class="language-cpp">#include "esp_camera.h"
#include <WiFi.h>
 
#define CAMERA_MODEL_AI_THINKER // Has PSRAM
 
#include "camera_pins.h"
 
const char* ssid = "REDACTED";
const char* password = "REDACTED";

void startCameraServer();
void setupLedFlash(int pin);
 
void setup() {
  pinMode(2, OUTPUT);
  Serial.begin(9600);
  while(!Serial);
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
  config.pixel_format = PIXFORMAT_JPEG; // for streaming
  //config.pixel_format = PIXFORMAT_RGB565; // for face detection/recognition
  config.grab_mode = CAMERA_GRAB_WHEN_EMPTY;
  config.fb_location = CAMERA_FB_IN_PSRAM;
  config.jpeg_quality = 12;
  config.fb_count = 1;
  
  // if PSRAM IC present, init with UXGA resolution and higher JPEG quality
  //                      for larger pre-allocated frame buffer.
  if(config.pixel_format == PIXFORMAT_JPEG){
    if(psramFound()){
      config.jpeg_quality = 10;
      config.fb_count = 2;
      config.grab_mode = CAMERA_GRAB_LATEST;
    } else {
      // Limit the frame size when PSRAM is not available
      config.frame_size = FRAMESIZE_SVGA;
      config.fb_location = CAMERA_FB_IN_DRAM;
    }
  } else {
    // Best option for face detection/recognition
    config.frame_size = FRAMESIZE_240X240;
#if CONFIG_IDF_TARGET_ESP32S3
    config.fb_count = 2;
#endif
  }
 
  // camera init
  esp_err_t err = esp_camera_init(&config);
  if (err != ESP_OK) {
    Serial.printf("Camera init failed with error 0x%x", err);
    return;
  }
 
  sensor_t * s = esp_camera_sensor_get();
  // initial sensors are flipped vertically and colors are a bit saturated
  if (s->id.PID == OV3660_PID) {
    s->set_vflip(s, 1); // flip it back
    s->set_brightness(s, 1); // up the brightness just a bit
    s->set_saturation(s, -2); // lower the saturation
  }
  // drop down frame size for higher initial frame rate
  if(config.pixel_format == PIXFORMAT_JPEG){
    s->set_framesize(s, FRAMESIZE_QVGA);
  }
 
// Setup LED FLash if LED pin is defined in camera_pins.h
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
 
  startCameraServer();
 
  Serial.print("Camera Ready! Use 'http://");
  Serial.print(WiFi.localIP());
  Serial.println("' to connect");
}
 
void loop() {
  // Do nothing. Everything is done in another task by the web server
  delay(10000);
}</code></pre>

While the entire project contains other library and control files, this program acts as the Command pattern. All files can be downloaded under <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week11/week11Downloads/">**this week's file download**</a>.

When uploading the code successfully to the ESP32 Cam board and executing it, the following message is printed to the Serial monitor.

<pre><code class="language-none">WiFi connected
Camera Ready! Use 'http://10.12.28.193' to connect</code></pre>

Upon connecting to the local address of 10.12.28.193, I access a local web server that allows me to control the camera. The main functions available on this page are to:

 - Configure camera resolution
 - Save a picture from current camera feed
 - Start and stop camera feed - this function is necessary to initialize the feed when starting up the camera and server for the first time, along with allowing me to reboot the camera in case of any issues.
 - Configure facial detection 
 - Configure facial recognition