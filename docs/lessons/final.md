# Final Project

## Brainstorming

My idea was to design a text to braille converter, which a blind person could use by moving the device over a page of text to convert it into braille. The braille translation of the english text would then be represented via a series of up/down pins which the user could use to interpret the information. The device was to be a rectangular box that would use an internal camera to interpret and OCR text, which could then be translated into braille and displayed via a series of servo motors pushing up metal rods on the top of the box. The pins would be in groups of six, each group representing a single braille character.

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

I then mapped out relevant weeks of Fab Academy:

 - Week 2: CAD - Designing the 3D printed structure
 - Week 4: Embedded Programming - Programming microcontrollers
 - Week 5: 3D Printing - Printing custom components
 - Week 6: Electronics - PCB design
 - Week 7: CNC - Creating a magnet, board making, internal wiring organization
 - Week 8: Electronics Production - Hooking up microcontrollers with magnets
 - Week 9: Output Devices - Board work
 - Week 11: Input Devices - Camera OCR work and hooking up to microcontroller
 - Week 12: Molding - Alternate option to mold the box
 - Week 13: Networking - Use a online libary/package for braille conversion
 - Week 14: Interfacing - Use Processing to flesh out the internal connections
 - Week 15: Wildcard - Electromagnets experimentation

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

### Bill of Materials

<div style="text-align: center"> 
<iframe src="https://docs.google.com/spreadsheets/d/e/2PACX-1vQlIJdCFYQU6-XJm1FrXhk5twaGxpRf5jiNvo1Z9Wf0MkVefTB23N4_w5QmfgFJcqXeWUzttINugkhU/pubhtml?widget=true&amp;chrome=false&amp;headers=false" frameborder="0" width="250%" height="300" scrolling="no"></iframe>
</div>

Note: It seems absurd that the electromagnets are so expensive, I will be looking for cheaper alternatives.

### Schedule for Completion

I created a Gantt chart to map out when I could finish tasks for my final.

<center>
<img src="../../pics/final/midterm/gantt.jpg" width="700"/>
</center>

As is shown in the chart, most of my remaining work can be during a week that corresponds with the task at hand. In the cases that don't, I plan to spend extra time completing those tasks during less intensive weeks.