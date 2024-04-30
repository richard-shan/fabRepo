# Networking

I decided to use this week's assignment to further develop my final project. I thus decided to develop two programs. One program would be uploaded to the ESP32CAM, which listens and waits for a remote connection via its local IP address on a common WiFi network. The ESP32 will then take a picture when prompted by the remote connection then send the data of the image over Websocket HTTP to a receiving device. The second program is a python script running on a computer or a Raspberry Pi. This script initiates a Websocket connection to the IP of the ESP32CAM, and then receives the image, which is saved locally on the main device.

## ESP32CAM Board Programming

The following program is uploaded onto the ESP32 CAM Board through Arduino IDE. This program requires a lot of libraries and dependencies, which I individually downloaded from Github. The entire project folder including the code and libraries can be downloaded under <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week13/week13Downloads/">**this week's file downloads**</a>. This program is based off of the CameraWebServer example program from ESP32.

<pre><code class="language-cpp">#include "esp_camera.h"
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
}</code></pre>

This program connects the ESP32CAM to a local WiFi network. It then sets up and initializes the camera, and sets up the local IP connection. It then continuously waits for a web socket connection. When a connection is created, it prints the IP address of the connecting device. If the device sends an input of "capture", the camera will take a picture and send it via the network web socket connection to the connecting computer/Pi.

## Recipient Device Programming

This python program is ran on a computer, Raspberry Pi, or other similar device with storage and WiFi capabilities. This program acts as the trigger for the ESP32CAM to capture an image. Something interesting I noticed is that when running the standard CameraWebServer, there is a lot of lag and downtime with the camera. However, when querying for a single capture, there is low latency.

<pre><code class="language-python">import websocket
import time

def on_message(ws, message):
    with open('received_image.jpg', 'wb') as f:
        f.write(message)
    print("Received image and saved.")

def on_error(ws, error):
    print("Error:", error)

def on_close(ws, close_status_code, close_msg):
    print("### closed ###")
    print("Close status code:", close_status_code)
    print("Close message:", close_msg)

def on_open(ws):
    def run(*args):
        time.sleep(1)
        ws.send("capture")
        time.sleep(1)
        ws.close()
    import threading
    threading.Thread(target=run).start()

if __name__ == "__main__":
    websocket.enableTrace(True)
    ws = websocket.WebSocketApp("ws://10.12.28.193:81",
                                on_message=on_message,
                                on_error=on_error,
                                on_close=on_close)
    ws.on_open = on_open
    ws.run_forever()</code></pre>
    
    This program initiates a WebSocket connection from the running device to the ESP32CAM board, which is served on the local IP of 10.12.28.193 in my case. It then sends the "capture" message to the ESP which triggers the image capture. Upon receiving the image data back from the ESP32, it will save the file in the relative local root directory.