# SAMD11C Chip Programming

I used the Quentorres RP2040 Board that I made during Week 4 to flash the initial firmware to the SAMD11C chip. I went through multiple iterations of a SAMD11C board before settling on the board I would use. After flashing the firmware to allow the chip to be recognized as a SAMD11C chip by my laptop's USB port and the Arduino IDE, I wrote a simple LED blinking program to test the board. I then wrote a bare metal program and sent that to the chip.

## Board Selection

Originally, I wanted to use a SAMDino board as my target board to program the SAMD11C. However, this board was not the final target board that I actually programmed. Due to a rough milling job and questionable soldering, I would end up abandoning this board in favor of the Hello D11C board. The Hello D11C board is a simpler board and also does not require the use of a connector, which is another part that the SAMDino requires that could have issues.

## Work Distributions
We worked in small groups for this assignment. My group consisted of myself and Alana Duffy.

<center>
<table>
    <tr>
        <td><b>People</b></td>
        <td><b>Description</b></td>
    </tr>
    </tr>
        <tr>
        <td>Richard</td>
        <td>Documentation</td>
    </tr>
    </tr>
        <tr>
        <td>Richard</td>
        <td>SAMDino Board Milling</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>Hello D11C Board Milling</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>Hello SWD 10-4 Pin Connector Milling</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>Hello D11C Board & SAMDino Board Programming</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>RP2040 Programmer Setup</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>Flashing Firmware with EDBG</td>
    </tr>
    <tr>
        <td>Richard</td>
        <td>Bare Metal Programming</td>
    </tr>
    <tr>
        <td>Alana</td>
        <td>SAMDino Board Soldering</td>
    </tr>
    <tr>
        <td>Alana</td>
        <td>Hello D11C Board Soldering</td>
    </tr>
</table>
</center>

## SAMDino

I initially decided to create a <a href="https://fabacademy.org/2020/labs/leon/students/adrian-torres/samdino.html">**SAMDino board**</a> for the SAMD11C chip. I got the PNG from the SAMDino's website, which I then converted into a millable file. 

<center>
<img src="../../../pics/week6/samdinoSchem.jpg" alt="SAMDino schematic" width="450"/>
</center>

I then opened the NC file in Bantam, and milled the job. The milling job was pretty rough, which I found out afterwards was because I happened to choose a bit that was slightly chipped on the end. I only realized this when checking over the health of our bits with my instructor, Mr Dubick.

<center>
<img src="../../../pics/week6/oldBoardBlank.jpg" alt="SAMDino board" width="450"/>
</center>

I then started preparing the programming as my partner, Alana Duffy, began to solder the SAMDino board.

<center>
<img src="../../../pics/week6/solderedOldBoard.jpg" alt="Soldered SAMDino board" width="450"/>
</center>

To be able to program the chip from the RP2040, we needed to design a connector that would adapt from a 2x5 pin cable to a 2x2. The RP2040 outputs from 2x5 pins but the SAMDino board takes a 2x2 pin input. I first downloaded the PNG of the schematic from the Fab Academy Schedule website.

<center>
<img src="../../../pics/week6/connectorSchem.jpg" alt="Connector Schematic" width="250"/>
</center>

I sent the schematic to Bantam Tools and ran the milling job. I milled two connector boards as I was going to give one of the boards to another group that was also dealing with the SAMD11C chip.

<center>
<video muted width="300" height="500" controls><source src="../../../pics/week6/newBoardMilling.mp4" type="video/mp4" /></video>
</center>

Alana then soldered the components.

<center>
<img src="../../../pics/week6/connectorComplete.jpg" alt="Connector Board completed" width="250"/>
</center>

I then took a careful look at the schematic of both the connector board and the SAMDino board to ensure that I wired the GND, RST, D10, and CLK pins to their respective corresponding pins on each board.

<center>
<img src="../../../pics/week6/connectedOldBoard.jpg" alt="Wired connector with SAMDino Board" width="250"/>
</center>

When I tried to program the board however, I encountered a multitude of errors. After around 5 hours of debugging, I ended up changing the SAMD11C board from the SAMDino to the Hello Board as it was a lot simpler.

## SAMD11C Hello Board

After deciding to switch to the Hello Board, I first downloaded the PNG of the schematic from the Fab Academy schedule.

<center>
<img src="../../../pics/week6/connectedOldBoard.jpg" alt="Wired connector with SAMDino Board" width="250"/>
</center>

I then turned the PNG into a NC millable file and sent it to Bantam. I milled the board using a 1/32" bit and a 1/64" bit.

<center>
<img src="../../../pics/week6/newBoardBlank.jpg" alt="Milled Hello Board" width="250"/>
</center>

Alana soldered as usual.

<center>
<img src="../../../pics/week6/solderedNewBoard.jpg" alt="Soldered Hello Board" width="250"/>
</center>

The board is now ready for programming!

## Flashing Firmware 

I first downloaded the <a href="https://taradov.com/bin/edbg/61/edbg-windows-r61.exe">**latest Windows version**</a> of EDBG from the binary releases found <a href="https://taradov.com/bin/edbg/">**latest Windows version**</a>.

