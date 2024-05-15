# Computer Vision

For wildcard week, I wanted to continue working towards my final project. Specifically, this week I wanted to build the process that would extract text from an image taken by the ESP32CAM.

## Raspberry Pi

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
<img src="../../../pics/week15/setup.jpg" width="500"/>
</center>

The ESP32CAM is pointed towards a paper with the words "Hello World!". In the right side of the picture, the Raspberry Pi which is running the code is visible along with the display. Upon running the program on the Pi's terminal, the ESP32CAM takes a picture and transmits it to the Pi, which then uses tesseract to perform OCR on it and prints out the extracted text.

<center>
<video width="550" height="300" controls><source src="../../../pics/week15/pi.mp4" type="video/mp4" /></video>
</center>

## Raspberry Pi Pico-W

The OCR works completely fine on a Raspberry Pi, but I wanted to try to perform it in a Pico as to work with computer vision in a more embedded sense. This is a lot more difficult as the Pico is far more low-level and is not an actual computer, unlike the Pi. Additionally, the Pico has very limited memory space and thus cannot even store an image. Finally, the Pico runs MicroPython which, while similar to Python, has a few key differences.

I first adapted the original code on the Pi to MicroPython and tried it out on the Pico.

```py
import network
import urequests
import ujson
import machine
import time

ssid = "REDACTED"
password = "REDACTED"

url = 'http://10.12.28.193/capture'

def connect_to_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    while not wlan.isconnected():
        pass
    print('Connected to Wi-Fi')

def main():
    connect_to_wifi()
    response = urequests.get(url)
    image = response.content
    status = response.status_code
    print("done")
    print(status)
    
if __name__ == '__main__':
    main()
    img_resp = urllib.request.urlopen(url)
    imgnp = np.array(bytearray(img_resp.read()), dtype=np.uint8)
    frame = cv2.imdecode(imgnp, -1)

    text = pytesseract.image_to_string(frame, config='--psm 6')

    print("Extracted Text:", text)
    time.sleep(1)
```

Unfortunately, this immediately led to a bunch of errors, along with generally not working. Namely, tesseract and openCV don't support MicroPython, and the Pico's memory is so small that it cannot handle storing an image. I decided to first work on the memory problem, then figure out a method to perform OCR.

Since the Pico can't handle processing the large image, I first created a custom handler on the ESP32CAM that would function similarly to the capture handler, but it would return a greyscale image that was only 20 by 20 pixels.

```cpp
static esp_err_t tiny_handler(httpd_req_t *req)
{
    camera_fb_t *fb = esp_camera_fb_get();
    if (!fb) {
        log_e("Camera capture failed");
        httpd_resp_send_500(req);
        return ESP_FAIL;
    }

    const int resize_width = 20;
    const int resize_height = 20;

    // Allocate memory for the small grayscale image
    uint8_t *small_img = (uint8_t *)malloc(resize_width * resize_height);
    if (!small_img) {
        log_e("Failed to allocate memory for small image");
        esp_camera_fb_return(fb);
        httpd_resp_send_500(req);
        return ESP_FAIL;
    }

    // Convert to grayscale and resize using simple nearest neighbor
    for (int y = 0; y < resize_height; ++y) {
        for (int x = 0; x < resize_width; ++x) {
            int orig_x = x * fb->width / resize_width;
            int orig_y = y * fb->height / resize_height;
            uint8_t *p = fb->buf + (orig_y * fb->width + orig_x) * 2; // Assuming the format is RGB565

            // Extract RGB components
            uint16_t color = (*p << 8) | *(p + 1);
            uint8_t r = (color >> 8) & 0xF8;
            uint8_t g = (color >> 3) & 0xFC;
            uint8_t b = (color << 3) & 0xF8;

            // Convert RGB to grayscale
            small_img[y * resize_width + x] = (r * 30 + g * 59 + b * 11) / 100; // Weights from the luminance calculation
        }
    }

    esp_camera_fb_return(fb);

    // Convert the grayscale image to JPEG
    uint8_t *jpeg_buf = NULL;
    size_t jpeg_len = 0;
    bool converted = fmt2jpg(small_img, resize_width * resize_height, resize_width, resize_height, PIXFORMAT_GRAYSCALE, 90, &jpeg_buf, &jpeg_len);
    free(small_img);

    if (!converted) {
        log_e("JPEG conversion failed");
        httpd_resp_send_500(req);
        return ESP_FAIL;
    }

    httpd_resp_set_type(req, "image/jpeg");
    httpd_resp_set_hdr(req, "Content-Disposition", "inline; filename=small_capture.jpg");
    httpd_resp_set_hdr(req, "Access-Control-Allow-Origin", "*");

    esp_err_t res = httpd_resp_send(req, (const char *)jpeg_buf, jpeg_len);
    free(jpeg_buf); // Free JPEG buffer after sending

    return res;
}
```

The above handler allows the accessing of a smaller-sized black and white compressed image, which decreases the net size of the image. I modified the Pico code to now access the black and white image.

```py
import network
import urequests
import ujson
import machine
import time

ssid = "REDACTED"
password = "REDACTED"

url = 'http://10.12.28.193/tiny'


def connect_to_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    while not wlan.isconnected():
        pass
    print('Connected to Wi-Fi')

def main():
    connect_to_wifi()
    response = urequests.get(url)
    image = response.content
    status = response.status_code
    print("done")
    print(status)

if __name__ == '__main__':
    main()
    img_resp = urllib.request.urlopen(url)
    imgnp = np.array(bytearray(img_resp.read()), dtype=np.uint8)
    frame = cv2.imdecode(imgnp, -1)

    text = pytesseract.image_to_string(frame, config='--psm 6')

    print("Extracted Text:", text)
    time.sleep(1)
```

Along with the library issues, the Pico still can't store the image despite its massively reduced size. In further testing, I even changed the ESP32 handler's pixel compression values to return a 5x5 pixel image, 2x2 pixel image, and even a 1x1 pixel image, and all of these attempts still resulted in the Pico failing to process and store the image.

I next decided to send the data over to the Pico in chunks of 1024 bytes each, and incrementally write data. 

```py
import network
import urequests
import ujson
import machine
import time

ssid = "REDACTED"
password = "REDACTED"

url = 'http://10.12.28.193/tiny'

def connect_to_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    while not wlan.isconnected():
        pass
    print('Connected to Wi-Fi')

def stream_image(url, chunk_size=1024):
    connect_to_wifi()
    
    response = urequests.get(url, stream=True)
    status = response.status_code
    print(f"HTTP Status Code: {status}")
    
    if status == 200:
        image = bytearray()
        for chunk in response.iter_content(chunk_size):
            image.extend(chunk)
        print("Image streamed successfully")
        return image
    else:
        print("Failed to retrieve the image")
        return None

connect_to_wifi()
print(stream_image())
```

Unfortunately, this still didn't work. I thus concluded that the issue was inherent to the Pico itself and supposed that it physically couldn't store all of this data.

After doing some research on how to solve this problem, I realized that I could use a microSD card to store information. After doing some research, I found that I could use SPI to directly write to the SD card via Pico pins, and I don't need any additional hardware.

I first found a microSD to SD adapter and soldered on some male to male wire headers to each respective pin.

<center>
<img src="../../../pics/week15/solderedSD.jpg" width="500"/>
</center>

I then wrote a program to the Pico that would test connecting and writing to the SD card.

```py
import os
import machine
import sdcard
import uos

# Setup SPI bus
spi = machine.SPI(0, baudrate=4000000, polarity=0, phase=0, sck=machine.Pin(18), mosi=machine.Pin(19), miso=machine.Pin(16))

# Setup SD card
sd = sdcard.SDCard(spi, machine.Pin(5))  # CS pin
vfs = os.VfsFat(sd)
uos.mount(vfs, '/sd')

# Test writing to a file
with open('/sd/test.txt', 'w') as f:
    f.write('Hello, SD card!')

# Test reading from a file
with open('/sd/test.txt', 'r') as f:
    print(f.read())

# List files in the root of the SD card
print('Files on SD card:', os.listdir('/sd'))
```

After I ran the program, I disconnected the SD card from the Pico and checked its contents on my computer. The program ran successfully and I had a test.txt file with the hello message inside!

Now that I knew the SD card methodology worked, I created a PCB to adapt from microSD to SPI pinout with a spare SD card holder that I found in the lab.

<center>
<img src="../../../pics/week15/pcbSD.jpg" width="500"/>
</center>

However, even when using the SD card to store the image file, it still saved as 0 bytes and didn't contain any data.

<center>
<img src="../../../pics/week15/zeroBytes.jpg" width="500"/>
</center>

When experimenting with writing to the SD card, I found that the Pico worked fine with just text. As such, I thought that I might be able to have the Pico store the image as a base 64 string, and decode it to an image when processing.

To do this, I first created a new handler on the ESP32CAM that would have it return a base64 string of a capture of the camera feed when that handler is called.

```cpp
static esp_err_t jpg_base64_handler(httpd_req_t *req) {
    camera_fb_t *fb = esp_camera_fb_get();
    if (!fb) {
        Serial.println("Camera capture failed");
        httpd_resp_send_500(req);
        return ESP_FAIL;
    }

    // Encode the frame in base64
    String base64Image = base64::encode(fb->buf, fb->len);

    // Send the base64 encoded image
    httpd_resp_set_type(req, "text/plain");
    esp_err_t res = httpd_resp_send(req, base64Image.c_str(), base64Image.length());

    // Return the frame buffer
    esp_camera_fb_return(fb);

    return res;
}

[...]

httpd_uri_t base64_uri = {
        .uri = "/base64",
        .method = HTTP_GET,
        .handler = jpg_base64_handler,
        .user_ctx = NULL

[...]

httpd_register_uri_handler(camera_httpd, &base64_uri);
```

A base64 string containing the data of a single frame captured by the ESP32CAM looks something like this:

```

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 
```

On Tuesday, the day I was just about to give up on the Pico, OpenAI released its GPT-4o model, which supports multimodal inputs. I spent some time reading about 4o, but the feature relevant to this project was that it supported an API call with a parsed base64 string as an input! 

With some modifications to <a href="https://platform.openai.com/docs/guides/vision">**OpenAI's sample code**</a> and the help of GPT-4o itself, I created the following script that would grab the base64 data from the custom handler and parse it into the model. The prompt for the model is "You are a image to text OCR engine. Output the text you see in this image, and nothing else."

```py
import requests

# OpenAI API Key
api_key = "REDACTED"

# Function to get the base64 encoded image from ESP32CAM
def get_base64_image(url):
    response = requests.get(url)
    return response.text

# URL to the ESP32CAM base64 image endpoint
esp32cam_url = "http://10.12.28.193/base64"

# Getting the base64 string
base64_image = get_base64_image(esp32cam_url)

headers = {
  "Content-Type": "application/json",
  "Authorization": f"Bearer {api_key}"
}

payload = {
  "model": "gpt-4o",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "You are a image to text OCR engine. Output the text you see in this image, and nothing else."
        },
        {
          "type": "image_url",
          "image_url": {
            "url": f"data:image/jpeg;base64,{base64_image}"
          }
        }
      ]
    }
  ],
  "max_tokens": 300
}

response = requests.post("https://api.openai.com/v1/chat/completions", headers=headers, json=payload)

print(response.json())
```

With the same setup as the Raspberry Pi, where the ESP32CAM is pointed towards a paper with the words "Hello World!", I ran the script on my computer to test it out.

Upon running, the following is printed out:

<center>
<img src="../../../pics/week15/4oJson.jpg" width="750"/>
</center>


As is seen in the output, the prompt worked and the 4o model detected "Hello World!" as the text extract and stored it in 'content'. To just output the content, which is the actual result I want, I can modify the code a little.

```py
response = requests.post("https://api.openai.com/v1/chat/completions", headers=headers, json=payload)

content_string = response.json()['choices'][0]['message']['content']
print(content_string)
```

Here is the code just printing out the extracted OCR text.

<center>
<img src="../../../pics/week15/4oExtracted.jpg" width="750"/>
</center>

Since this script is written in regular Python, not MicroPython, I have to make a wrapper class for the Pico to execute it.

I first uploaded the main python script to the Pico's microSD card as get4o.py.

<center>
<img src="../../../pics/week15/4oSD.jpg" width="450"/>
</center>

With the help of ChatGPT, I then created the following program which I would upload to the Pico via Thonny. This program takes care of setting up the Pico before running the main program, connecting it to Wifi and initializing its connection with the SD card. Once the setup is complete, it will run the get4o.py program from the SD card.

```py
import os
import machine
import sdcard
import uos
import network
import time

ssid = "REDACTED"
password = "REDACTED"


def connect_to_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    while not wlan.isconnected():
        pass
    print('Connected to Wi-Fi')

connect_to_wifi()

# SPI configuration
spi = machine.SPI(0, baudrate=4000000, polarity=0, phase=0, sck=machine.Pin(18), mosi=machine.Pin(19), miso=machine.Pin(16))

# SD card configuration
sd = sdcard.SDCard(spi, machine.Pin(5))  # Adjust pin according to your CS pin
uos.mount(sd, "/sd")

os.chdir("/sd")

# Execute the script
try:
    with open("get4o.py", "r") as file:
        print("Opened get")
        exec(file.read())
except OSError as e:
    print("Failed to read script from SD card:", e)

uos.umount("/sd")
```

I then ran the code.

<center>
<img src="../../../pics/week15/onPico.jpg" width="500"/>
</center>

## Conclusion

I will likely use the Raspberry Pi OCR for my final project, but I will shift away from the pytesseract model and to the GPT-4o model as it seems to generally perform better. Because I need to do other advanced tasks in my final project more than just OCR, using the Raspberry Pi will be easier overall. I did, however, enjoy debugging with the Pico to find a way to perform OCR on a really limited-resource environment.