# SAMD11C Chip Programming

I used the Quentorres RP2040 Board that I made during Week 4 to flash the initial firmware to the SAMD11C chip. I went through multiple iterations of a SAMD11C board before settling on the board I would use. After flashing the firmware to allow the chip to be recognized as a SAMD11C chip by my laptop's USB port and the Arduino IDE, I wrote a simple LED blinking program to test the board. I then wrote a bare metal program and sent that to the chip.

## SAMD11C Board Designing

I initially decided to create a <a href="https://fabacademy.org/2020/labs/leon/students/adrian-torres/samdino.html">**SAMDino board**</a> for the SAMD11C chip. I got the PNG from the SAMDino's website, which I then converted into a millable file.

<center>
<img src="../../../pics/week6/samdinoSchem.jpg" alt="SAMDino schematic" width="450"/>
</center>
