# Programming

I was responsible for the entirety of our group's programming and software development, along with working with (and deciding upon) the Raspberry Pi. I also for selected, set up, and configured the microphone, keyboard, mouse, and touchscreen. I was also responsible for all of our software debugging and assisted in debugging the motors.

## Planning

The final program must be able to process a user's question through voice and display an answer on the actual ouija board by moving the magnet to corresponding letter locations.

The whole program for the board can be broken down into a few key tasks: receiving audio input from the microphone, converting audio to text using Whisper API, querying ChatGPT (OpenAI API) for a response to that same text, sending the generated text from the chip to the Arduino, and moving the magnet to a specified set of coordinates that correspond to each letter within the text string.

With these goals established, it directly follows that our chosen chip must be able to:

 - Connect to the internet to query APIs
 - Receive audio input from a USB microphone and/or keyboard
 - Use serial communication with an Arduino

As a personal goal, I wanted the entire machine to be self-contained and run without the need of any laptop. This self-imposed goal definitely made the entire process a lot more difficult, necessitating the following major changes (and many more minor ones!).

<center>
<table>
    <tr>
        <td><b>With laptop</b></td>
        <td><b>Without laptop</b></td>
    </tr>
    <tr>
        <td>Inputting coordinates into Arduino IDE's Serial Monitor</td>
        <td>Serial communication between Raspberry Pi and Arduino</td>
    </tr>
    <tr>
        <td>Using computer capabilities for API calls</td>
        <td>Chip connection to wireless network and querying APIs<td>
    </tr>
    <tr>
        <td>Builtin keyboard for text input</td>
        <td>Support generic USB keyboards for text input (not PS/2 specific)</td>
    </tr>
    <tr>
        <td>Builtin computer microphone for audio input</td>
        <td>Script to record sound through USB microphone and save file to a static local path for processing</td>
    </tr>
</table>
</center>

However, I'm glad that I decided to run the whole machine without a laptop. First off, it simply is cooler. Secondly, even with this extra challenge, I still spent a lot of time waiting for the mechanical part of the machine to be built and without this modification, I likely would have finished the code on the first or second day. Lastly, running the code without a laptop (albeit with a Raspberry Pi) taught me a lot of important skills on networking, serial communication, and managing inputs that will be of massive help in later weeks. 

## Finding a Chip

### ESP

I initially tried both an ESP32 and ESP8266 chip, but abandoned them due to issues connecting to a WiFi network. Although I know that these chips are capable of connecting to a wireless network, I wasn't able to do it after about an hour of work.

### Raspberry Pi Pico

My next (and more promising) chip was the Raspberry Pi Pico. I found a Pico with a built-in WiFi chip, which allowed me to connect to WiFi to query OpenAI. Using the Pico's TX/RX pins, I was also able to confirm serial communication with this chip. Additionally, using a Pi-family chip allowed me to easily integrate Python/MicroPython, which made querying OpenAI and Whisper APIs exponentially easier. The following code connects the Pico to WiFi, queries OpenAI, and sends the response to an Arduino having been connected through TX/RX pins.

<pre><code class="language-python">import network
import urequests
import ujson
import machine
import time

ssid = "NETWORKNAME"
password = "NETWORKPASSWORD"

api_key = 'USER API KEY'
prompt = 'PROMPT'
url = 'https://api.openai.com/v1/chat/completions'

def connect_to_wifi():
    wlan = network.WLAN(network.STA_IF)
    wlan.active(True)
    wlan.connect(ssid, password)

    while not wlan.isconnected():
        pass
    print('Connected to Wi-Fi')

def send_prompt_to_openai():
    headers = {
        'Authorization': 'Bearer ' + api_key,
        'Content-Type': 'application/json'
    }
    data = {
        'model': 'gpt-3.5-turbo',
        "messages": [{"role": "user", "content": "Answer the following query in an ominous manner, and keep your response to under 20 characters:" + prompt}],
        'max_tokens': 50
    }
    response = urequests.post(url, data=ujson.dumps(data), headers=headers)
    return response.json()

def main():
    connect_to_wifi()
    response = send_prompt_to_openai()
    response_text = response['choices'][0]['message']['content']
    print(response_text + "\n")
    arduinoify(response_text)

def arduinoify(response):
    uart1 = machine.UART(1, baudrate=9600, tx=4, rx=5)
    uart1.write(response)

if __name__ == '__main__':
    main()</code></pre>

However, the Pico had a critical flaw in that I couldn't easily integrate any USB-based input device - namely, in this case, a USB microphone or USB keyboard. After doing some research online, I discovered that in order to use a keyboard, I either needed to switch from using MicroPython to CircuitPython (which still only may or may not work) or to buy an entirely new keyboard with PS/2 compatibility, neither of which were realistic solutions.

### Raspberry Pi 4B

I was a little hesitant at first to use a Raspberry Pi, both because using it would require an entire redesign of my current code (connecting to OpenAI via completion model URL changed to querying via a completion prompt), and because the Raspberry Pi itself could qualify as a computer. However the Raspberry Pi just seemed to check all of the boxes, given that it has builtin WiFi connectivity, runs Python, has 4 USB ports for inputs (keyboard, microphone), and can plug directly into the Arduino, effectively eliminating the need for TX/RX pin wiring for serial communication.

However, changing to a Raspberry Pi required a few modifications:

 - Since the Raspberry Pi is more a computer than a chip (runs Linux), a code redesign is necessary. As mentioned earlier, I no longer used a URL endpoint to query OpenAI but rather assembled the entire prompt locally and sent it out to a model (either GPT3.5 or GPT4 - for the purposes of keeping my API costs low, I use GPT3.5 but there should be no significant difference given that all generated responses are soft capped at 20 characters.)
 - A screen. Once more, because the Pi resembles an actual computer, I have to actually execute and call a script on the Pi as opposed to loading a script onto it. I actually always wanted to add a screen to this project, but with a Pi, adding a screen allows me to run and view the status of the program from the actual machine. The screen also functions as the HDMI output of the Pi - for many hours, I was coding on a small 5 inch screen.

## Software Development

Now that I have decided on a chip and have a more clear idea of the entire system, it's time to start programming. Again, envision the final program as a conglomeration of several key tasks: 

 - Receiving audio input from the Microphone
 - Saving the audio file and converting to text (Whisper API)
 - Generating the oujia board's response to the user query (OpenAI API + some minor prompt engineering)
 - Processing the response text into movements for the gantries
 - Moving the gantries and therefore the magnet

### Audio Input

As I had little prior experience in both Linux and working with microphone input devices, I developed the following handy program that allowed me to view all connected audio devices along with some of their specs.

<pre><code class="language-python">import pyaudio

audio = pyaudio.PyAudio()

def print_device_info(device_index):
    device_info = audio.get_device_info_by_index(device_index)
    print(f"Device {device_index}: {device_info.get('name')}")
    print(f"  Input Channels: {device_info.get('maxInputChannels')}")
    print(f"  Output Channels: {device_info.get('maxOutputChannels')}")
    print(f"  Default Sample Rate: {device_info.get('defaultSampleRate')}")

num_devices = audio.get_device_count()

print(f"Found {num_devices} device(s)\n")

for i in range(0, num_devices):
    print_device_info(i)

audio.terminate()</code></pre>

As a demonstration, when ran on my computer, the above code yields the following output:

<pre><code class="language-none">Device 0: Microsoft Sound Mapper - Input
  Input Channels: 2
  Output Channels: 0
  Default Sample Rate: 44100.0
Device 1: Microphone Array (IntelÂ® Smart
  Input Channels: 4
  Output Channels: 0
  Default Sample Rate: 44100.0
Device 2: Microsoft Sound Mapper - Output
  Input Channels: 0
  Output Channels: 2
  Default Sample Rate: 44100.0

[...]

Device 19: PC Speaker (Realtek HD Audio 2nd output with SST)
  Input Channels: 2
  Output Channels: 0
  Default Sample Rate: 48000.0
Device 20: Stereo Mix (Realtek HD Audio Stereo input)
  Input Channels: 2
  Output Channels: 0
  Default Sample Rate: 48000.0</code></pre>

As is shown in the printout, this script finds the input channels, output channels, and default sample rate of any connected audio device, along with its device number.

When running the diagnostic script on the Raspberry Pi, I learn that the USB microphone is recognized as Device #2, with 2 audio input channels and a default sample rate of 44100.0. These specifications are important for recording audio.

Now that the device specifications are cleared up, the actual coding can begin. The following script records 10 seconds of audio and saves it to "output.wav".

<pre><code class="language-python">import pyaudio
import wave

# Audio recording parameters
FORMAT = pyaudio.paInt16  # Audio format (16-bit PCM)
CHANNELS = 2              # The microphone has 2 audio input channels
RATE = 22050              # Reduced sampling rate from default of 44100 to minimize overflow errors (they are bad)
CHUNK = 2048              # Increased buffer size to minimize overflow errors (they are bad)
RECORD_SECONDS = 10        # Duration of recording
WAVE_OUTPUT_FILENAME = "output.wav"  # Output filename

def record_audio():
    audio = pyaudio.PyAudio()

    # Open stream
    stream = audio.open(format=FORMAT, channels=CHANNELS,
                        rate=RATE, input=True, input_device_index=2, # Device index of 2 because the microphone is recognized as Device #2 
                        frames_per_buffer=CHUNK)

    print("Recording...")
    frames = []

    # Record for specified number of seconds
    try:
        for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
            try:
                data = stream.read(CHUNK)
                frames.append(data)
            except IOError as e:
                if e.errno == pyaudio.paInputOverflowed:
                    print(f"Overflow at iteration {i}. Continuing...")
                    time.sleep(0.1) 
    except Exception as e:
        print(f"Error during recording: {e}")

    print("Recording finished.")
    stream.stop_stream()
    stream.close()
    audio.terminate()

    # Save the recorded data as a WAV file
    try:
        with wave.open(WAVE_OUTPUT_FILENAME, 'wb') as wf:
            wf.setnchannels(CHANNELS)
            wf.setsampwidth(audio.get_sample_size(FORMAT))
            wf.setframerate(RATE)
            wf.writeframes(b''.join(frames))
    except Exception as e:
        print(f"Error saving WAV file: {e}")</code></pre>

### Audio to Text

Converting audio to text is surprisingly easy, although the process can take upwards of 20 seconds for a short clip.

<pre><code class="language-python">import whisper

def transcribe_audio(file_path):
    model = whisper.load_model("base")
    result = model.transcribe(file_path)
    return result["text"]</code></pre>

Side note: the code's implementation is set up so that every time the recording occurs, it overwrites the previous recording in the output.wav file. This means that there is no wasted storage on old audio clips, however, this could easily be tweaked if you are trying to implement this with saving.

### Generative Text Response

Now that we have extracted text from the short microphone audio clip, we can send that text to OpenAI's API to generate a response. The following function queries GPT3.5 with a parsed text prompt and returns the output of the query. This process is surprisingly fast.

<pre><code class="language-python">from openai import OpenAI

def call_openai_api(prompt):
    client = OpenAI(api_key='REDACTED') 
    completion = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You are a helpful assistant trapped in a ouija board and are a ghost."},
            {"role": "user", "content": "Answer the following query in an ominous tone and under 25 characters: " + prompt}
        ]
    )
    return completion.choices[0].message.content
</code></pre>

### Serial Connection

This code snippet, in isolation, initializes a new serial connection. Since the Pi is wired straight to the Arduino's USB port, writing to the serial here is equivalent to typing something into the Arduino IDE's Serial Monitor. Obviously, the code is not used in isolation in the actual program.

<pre><code class="language-python">from serial import Serial

ser = Serial('/dev/ttyUSB0', 115200, timeout = 1)
ser.write(("Whatever you want to write").encode())
</code></pre>

The ```lsusb``` builtin command on linux displays all connected USB devices. When unplugging the Arduino and rerunning ```lsusb```, I can see that the ttyUSB0 device has disappeared. Therefore, the Arduino is the tty0USB device.

The ```.encode()``` function is important because it turns the string into bytes before sending it through serial. Otherwise, Arduino cannot interpret the string.

### Moving the Magnet

To move the magnet based on each character in the text string, I first need to find physical positions of each character and map them to that character. This process took about 20 minutes to do, as it was just some manual labor.

<pre><code class="language-python">alphabet_dict = {chr(65 + i): (0, 0) for i in range(26)}

alphabet_dict['A'] = (1, 20)
alphabet_dict['B'] = (5, 25)
alphabet_dict['C'] = (10, 27)
alphabet_dict['D'] = (16, 28)
alphabet_dict['E'] = (20, 30)
alphabet_dict['F'] = (27, 30)
alphabet_dict['G'] = (34, 30)
alphabet_dict['H'] = (37, 31)
alphabet_dict['I'] = (41, 31)
alphabet_dict['J'] = (44, 28)
alphabet_dict['K'] = (51, 27)
alphabet_dict['L'] = (57, 25)
alphabet_dict['M'] = (64, 22)
alphabet_dict['N'] = (0, 10)
alphabet_dict['O'] = (3, 12)
alphabet_dict['P'] = (9, 17)
alphabet_dict['Q'] = (14, 20)
alphabet_dict['R'] = (19, 22)
alphabet_dict['S'] = (24, 23)
alphabet_dict['T'] = (30, 23)
alphabet_dict['U'] = (36, 23)
alphabet_dict['V'] = (41, 21)
alphabet_dict['W'] = (47, 20)
alphabet_dict['X'] = (55, 17)
alphabet_dict['Y'] = (59, 14)
alphabet_dict['Z'] = (64, 10)
alphabet_dict['1'] = (9, 5)
alphabet_dict['2'] = (15, 5)
alphabet_dict['3'] = (19, 5)
alphabet_dict['4'] = (24, 5)
alphabet_dict['5'] = (29, 5)
alphabet_dict['6'] = (34, 5)
alphabet_dict['7'] = (39, 5)
alphabet_dict['8'] = (43, 5)
alphabet_dict['9'] = (48, 5)
alphabet_dict['0'] = (53, 5)
alphabet_dict['.'] = (24, 0)
alphabet_dict[','] = (40, 0)
alphabet_dict[' '] = (33, 13)</code></pre>

The dictionary maps the physical coordinate values of each letter on the actual engraved ouija board to the character in the text string. Now to actually move the magnet to the right location:

<pre><code class="language-python">def move(letter):
    time.sleep(4)
    x, y = alphabet_dict[letter.upper()]
    ser.write(("g1x" + str(x) + "y" + str(y) + "f2000" + "\n").encode());
    time.sleep(4)</code></pre>

The command format for moving the gantries is shown in the ser.write command. For example, writing ```g1x40y25f500\n``` would move the magnet to (40, 25) at a frequency (speed) of 500.

The newline character is important in actually sending the command across. Without it, this is analogous to typing something into the Arduino IDE's Serial Monitor and not clicking the enter button.

With the move function defined, a simple for each loop that iterates through the string that we will display on the ouija board is all I need.

The delays are important so that the gantries actually have time to move. These values can be further optimized - increasing or decreasing them depending on how much time you want the magnet to stay on each letter for and how much leeway you want to give the machine.

<pre><code class="language-python">for char in response:
    move(char)</code></pre>

## Aggregation

To tie up all the individual parts:

<pre><code class="language-python">import pyaudio
import wave
import time
import whisper
import os
import curses
from openai import OpenAI
from serial import Serial


alphabet_dict = {chr(65 + i): (0, 0) for i in range(26)}

alphabet_dict['A'] = (1, 20)
alphabet_dict['B'] = (5, 25)
alphabet_dict['C'] = (10, 27)
alphabet_dict['D'] = (16, 28)
alphabet_dict['E'] = (20, 30)
alphabet_dict['F'] = (27, 30)
alphabet_dict['G'] = (34, 30)
alphabet_dict['H'] = (37, 31)
alphabet_dict['I'] = (41, 31)
alphabet_dict['J'] = (44, 28)
alphabet_dict['K'] = (51, 27)
alphabet_dict['L'] = (57, 25)
alphabet_dict['M'] = (64, 22)
alphabet_dict['N'] = (0, 10)
alphabet_dict['O'] = (3, 12)
alphabet_dict['P'] = (9, 17)
alphabet_dict['Q'] = (14, 20)
alphabet_dict['R'] = (19, 22)
alphabet_dict['S'] = (24, 23)
alphabet_dict['T'] = (30, 23)
alphabet_dict['U'] = (36, 23)
alphabet_dict['V'] = (41, 21)
alphabet_dict['W'] = (47, 20)
alphabet_dict['X'] = (55, 17)
alphabet_dict['Y'] = (59, 14)
alphabet_dict['Z'] = (64, 10)
alphabet_dict['1'] = (9, 5)
alphabet_dict['2'] = (15, 5)
alphabet_dict['3'] = (19, 5)
alphabet_dict['4'] = (24, 5)
alphabet_dict['5'] = (29, 5)
alphabet_dict['6'] = (34, 5)
alphabet_dict['7'] = (39, 5)
alphabet_dict['8'] = (43, 5)
alphabet_dict['9'] = (48, 5)
alphabet_dict['0'] = (53, 5)
alphabet_dict['.'] = (24, 0)
alphabet_dict[','] = (40, 0)
alphabet_dict[' '] = (33, 13)

ser = Serial('/dev/ttyUSB0', 115200, timeout = 1)

FORMAT = pyaudio.paInt16
CHANNELS = 2
RATE = 22050
CHUNK = 2048
RECORD_SECONDS = 10
WAVE_OUTPUT_FILENAME = "output.wav"

def record_audio():
    audio = pyaudio.PyAudio()

    stream = audio.open(format=FORMAT, channels=CHANNELS,
                        rate=RATE, input=True, input_device_index=2,
                        frames_per_buffer=CHUNK)

    print("Recording...")
    frames = []

    try:
        for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
            try:
                data = stream.read(CHUNK)
                frames.append(data)
            except IOError as e:
                if e.errno == pyaudio.paInputOverflowed:
                    print(f"Overflow at iteration {i}. Continuing...")
                    time.sleep(0.1)
    except Exception as e:
        print(f"Error during recording: {e}")

    print("Recording finished.")
    stream.stop_stream()
    stream.close()
    audio.terminate()

    try:
        with wave.open(WAVE_OUTPUT_FILENAME, 'wb') as wf:
            wf.setnchannels(CHANNELS)
            wf.setsampwidth(audio.get_sample_size(FORMAT))
            wf.setframerate(RATE)
            wf.writeframes(b''.join(frames))
    except Exception as e:
        print(f"Error saving WAV file: {e}")

def transcribe_audio(file_path):
    model = whisper.load_model("base")
    result = model.transcribe(file_path)
    return result["text"]

def move(letter):
    time.sleep(1.5)
    x, y = alphabet_dict[letter.upper()]
    ser.write(("g1x" + str(x) + "y" + str(y) + "f2000" + "\n").encode());
    time.sleep(1.5)

def call_openai_api(prompt):
    client = OpenAI(api_key='REDACTED')
    completion = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You are a helpful assistant trapped in a ouija board and are a ghost."},
            {"role": "user", "content": "Answer the following query in an ominous tone and under 20 characters: " + prompt}
        ]
    )
    return completion.choices[0].message.content

def main():
    record_audio()

    transcription = transcribe_audio(WAVE_OUTPUT_FILENAME)
    print("Transcribed: ", transcription)

    response = call_openai_api(transcription)
   
    print("Response from OpenAI:\n", response)
   
    for char in response:
        move(char)
       
    t_end = time.time() + 10
    while time.time() < t_end:
        ser.write(("g1x0y0f500\n").encode());
   
if __name__ == '__main__':
    main()</code></pre>

The only discrepancy between this code and the sum of each individual task is the ```g1x0y0f500``` command at the end, which brings the machine back to (0, 0). An interesting phenomenon occurs - that when the program ends, the machine will set its current point as (0, 0). As such, after the program finishes, it is hard coded to return to the true home point so it is ready for the next query.