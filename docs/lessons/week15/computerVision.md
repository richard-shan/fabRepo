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

OCR works completely fine on a Raspberry Pi, but I wanted to try to perform it in a Pico. This is a lot more difficult as the Pico is far more low-level and is not an actual computer, unlike the Pi. Additionally, the Pico has very limited memory space and thus cannot even store an image. Finally, the Pico runs MicroPython which, while similar to Python, has a few key differences.

I first adapted the code on the Pi to MicroPython and tried it out.

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

## GPT-4o

fun test, won't use bc its external dependency and needs money per tokens

## Nanonets