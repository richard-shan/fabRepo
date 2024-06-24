# Final Project

## Overview

I want to create a 3x2 solenoid array that can display braille characters by pushing solenoids up and down to create dots. This solenoid array will be connected to a Raspberry Pi, which in turn will be connected to an ESP32CAM. The camera will take a picture of a page of text, then perform OCR (optical character recognition) to extract a string of text from the image. That string of text will be converted to braille, which will be displayed on the solenoid array by flashing each character for 1 second at a time. This device will essentially allow for live-time conversion of any text into braille, which I hope will increase accessibility to books and the like.

## Brainstorming Process

### Initial Thoughts

My idea was to design a text to braille converter, which a blind person could use by moving the device over a page of text to convert it into braille. The braille translation of the english text would then be represented via a series of up/down pins which the user could use to interpret the information. The device was to be a rectangular box that would use an internal camera to interpret and OCR text, which could then be translated into braille and displayed via a series of servo motors pushing up metal rods on the top of the box. The pins would be in groups of six, each group representing a single braille character.

However, I talked to <a href="https://fabacademy.org/2023/labs/charlotte/students/stuart-christhilf/">**Stuart Christhilf**</a> who had thought of a similar mechanism for his initial final project. He originally planned to create a dynamic clock to display the time using blocks of wood that acould be pushed out or pulled back via servos. However, when building his project, he realized that fitting so many servos into such a small space was completely unfeasible and warned me from doing the same. My initial design is shown in the following image:

<center>
<img src="../../../pics/week1/initialDesign.jpg" alt="Initial design" width="450"/>
</center>
<br>

I then decided to use electromagnets for my pins, instead of a servo. The pins themselves would be a small magnetic rod sitting on top of an electromagnet. The small electromagnet could be powered on and off via a microcontroller. When the electromagnet was off, the pin would simply rest on top of the electromagnet, and the pin would be flush against the top of the board, forming the down position of the pin. If the pin needed to pop up, the microcontroller would power the electromagnet which would then emit a repelling magnetic charge. That magnetic force would then repel the pin slightly upwards, forming the up position of the pin. To represent a braille character, the microcontroller would push the specific pins into the up position that together would form the 6-dot pattern of the character.

I also decided to move the camera out of the box. That would allow for more simple wiring and internal organization of the box, and allow the operator to more easily use the device. Moving the camera out means that the user would only need to move a small camera container across the page of text, instead of dragging the entire device. Here is my modified design:

<center>
<img src="../../../pics/week1/modifiedDesign.jpg" alt="Modified design" width="700"/>
</center>
<br>

### Significant Changes

Although a large part of my project remains the same, I've changed some aspects of my project. Namely, I've decided to use a Raspberry Pi as a central controller and connect it to 5 separate ATTiny412 chips, which will each be responsible for controlling 6 electromagnets to represent 1 braille character. Each ATTiny412 and 6 electromagnet setup will be on its own PCB, and receive data from the controlling Raspberry Pi. Additionally, I decided to create an elevated case for the ESP32 camera so that the image would have a better angle and thus an easier time being processed for OCR, and so that more light could come into the camera lens from the unobstructed sides. Lastly, I decided I wanted to wirelessly transmit data from the ESP32 camera to the Raspberry Pi for processing. I worked with both serial communication and WiFi connectivity in previous weeks so I hope to sum it all together and wirelessly transmit data between these two controllers.

Here is an updated system diagram which maps out all the parts of my project.

<center>
<img src="../../../pics/final/midterm/systemDiagram.jpg" width="700"/>
</center>

### Feasibility

However, after doing research, I realized that having 30 solenoids would be unfeasible. Instead, I decided to scale my project down to just having 6 solenoids, as this would still accomplish the mission of displaying braille for a reader. I would then flash each braille character for 1 second on the 6 solenoid array. This change allows me to worry less about power budget and ensures that I have a ready final project on my presentation date.

### Bill of Materials

<div style="text-align: center"> 
<iframe src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQlIJdCFYQU6-XJm1FrXhk5twaGxpRf5jiNvo1Z9Wf0MkVefTB23N4_w5QmfgFJcqXeWUzttINugkhU/pubhtml?widget=true&amp;chrome=false&amp;headers=false" frameborder="0" width="250%" height="300" scrolling="no"></iframe>
</div>


### Schedule for Completion

I previously had created a Gantt chart around week 10 to map out when I could finish tasks for my final.

<center>
<img src="../../../pics/final/midterm/gantt.jpg" width="700"/>
</center>

This chart still holds true, albeit I have decided that instead of controlling multiple ATTiny412s, I now want to use a single ATTiny1614. However, the plan as a whole is proceeding nicely.

## Progress Check

 - ☐ design and print a PCB
 - ☑ test controlling 6 solenoids through single ATTiny1614
 - ☑ develop Bluetooth communication between ESP32 and RPi
 - ☑ text to braille conversion
 - ☑ research processing capabilities of ESP32 and decide whether to process images on the ESP32 or on the RPi
 - ☐ assemble the project
 - ☐ transistor debugging
 - ☑ design a custom case
 - ☑ integrate with WiFi
 - ☑ test serial communication
 - ☑ find Python OCR library (pytesseract)
 - ☑ find Python library to convert text to braille (https://github.com/AaditT/braille - possibly used for OCR too?)

I am finished with all essential parts of my project except the transistor control, which has been giving me some trouble. The OCR and communication portions are all working, and I just need to integrate the transistor control once that is ready.

## Processes Used

 - CAD for ESP32CAM case, Raspberry Pi case, solenoid array case
 - 3D Printing for ESP32CAM case, Raspberry Pi case, solenoid array case
 - Electronics Design for designing the ATTiny1614 solenoid control board
 - Electronics Production for milling and soldering the solenoid control board
 - Embedded Programming for controlling the solenoids
 - Networking for communication between ESP32CAM, RPi, and ATTiny1614
 - System Integration for assembly of solenoid box and integration of parts
 - Python script for OCR API calls/serial comm
 - Arduino for ATTiny1614 braille character/solenoid array mapping
 - Arduino for ESP32CAM configuration
 - Networking and Interfacing for ESP32CAM custom event handler creation

## Evaluation

My project is considered a success if it can:

 - ☑ Accurately extract text from a live image feed
 - ☑ Map the text to braille
 - ☑ Display the braille on the solenoid array