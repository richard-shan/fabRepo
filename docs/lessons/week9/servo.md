# Servo

## Breadboard

I decided that I wanted to control a servo as my output device. At first, I wanted to use the <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week8/design/">**board**</a> that I created in Week 8. However, I didn't create any power or ground external connector pins that would allow me to connect a servo, a critical flaw of the board that I didn't realize in week 8. However, I still used this board to test that my understanding and wiring for the servo was correct by using a breadboard. The breadboard allowed me to create additional pin connections for power and ground.

<center>
<img src="../../../pics/week9/breadboardSetup.jpg" alt="Breadboard setup" width="450"/>
</center>

In this setup, I have connected the VCC and GND output pins of the Quentorres board to a breadboard, which is then connected to the VCC and GND pins of my ATTiny412 board. This effectively creates more pin openings that I can connect to VCC and GND. The UPDI is still wired directly from the Quentorres to the ATTiny412 board because I do not need to connect anything else to it. The servo data pin itself is pin 3 of the ATTiny412, or pin 1 in Arduino. 

After doing some research on how servos worked, I came up with this code. This is a servo sweep program.

<pre><code class="language-cpp">#include &ltServo.h&gt

Servo servo;

void setup() {
  servo.attach(1);  // attaches the servo on pin 1 to the servo object
}

void loop() {
  for (int pos = 0; pos <= 180; pos += 1) { 
    // goes from 0 degrees to 180 degrees in steps of 1 degree
    servo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
  for (int pos = 180; pos >= 0; pos -= 1) { 
    // goes from 180 degrees to 0 degrees
    servo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
}</code></pre>

Here is a video of the servo working through the breadboard.

<center>
<video width="500" height="300" controls><source src="../../../pics/week9/breadboardOn.mp4" type="video/mp4" /></video>
</center>

## PCB

Because breadboards are uncouth, I decided to design a really simple ATTiny412 board that would control the servo and be programmed through the Quentorres. This board is incredibly simple and was created for one sole purpose: spin the servo. I include a power ground capacitor, a power indicator LED with a resistor, and a set of connector headers for VCC, GND, and DATA that would control the servo. Here is the schematic.

<center>
<img src="../../../pics/week9/schem.jpg" alt="PCB Schematic" width="700"/>
</center>

Here is my PCB design after routing the traces.

<center>
<img src="../../../pics/week9/pcb.jpg" alt="PCB" width="600"/>
</center>

I then exported the design as a GBR file and milled it. Here is my milled board with the ATTiny412 soldered on.

<center>
<img src="../../../pics/week9/blankishBoard.jpg" alt="Blank board with ATTiny412 soldered on" width="500"/>
</center>

Here is the board after I soldered on the remaining capacitor, resistor, LED, and conn pin headers.

<center>
<img src="../../../pics/week9/solderedBoard.jpg" alt="Fully soldered PCB" width="600"/>
</center>

I then connected the board to the RP2040 board via the adapter I made <a href="https://fabacademy.org/2024/labs/charlotte/students/richard-shan/lessons/week8/soldering/">**previously**</a>. When I plugged everything in, the power indication LED turned on - a good sign!

<center>
<img src="../../../pics/week9/connected.jpg" alt="Connected boards" width="600"/>
</center>

I then slightly modified the original breadboard program to use ATTiny412 pin 5, or Arduino pin 3.

<pre><code class="language-cpp">#include &ltServo.h&gt

Servo servo;

void setup() {
  servo.attach(3);  // attaches the servo on pin 3 to the servo object
}

void loop() {
  for (int pos = 0; pos <= 180; pos += 1) { 
    // goes from 0 degrees to 180 degrees in steps of 1 degree
    servo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
  for (int pos = 180; pos >= 0; pos -= 1) { 
    // goes from 180 degrees to 0 degrees
    servo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                       // waits 15ms for the servo to reach the position
  }
}</code></pre>

Here is the servo turning, this time with no breadboard.

<center>
<video width="500" height="300" controls><source src="../../../pics/week9/on.mp4" type="video/mp4" /></video>
</center>

I then attached a component to the top of the servo gear so that in the video, its movement can be seen more explicitly.

<center>
<video width="500" height="300" controls><source src="../../../pics/week9/onWithTop.mp4" type="video/mp4" /></video>
</center>