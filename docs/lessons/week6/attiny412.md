# ATTiny412 Chip Programming

After I finished work with my SAMD11C board, I swapped my SAMD11C chip board with a fellow classmate for his ATTiny412 chip board. As such, I was not responsible for milling, soldering, or setting up the board. More comprehensive documentation on JTAG, which was used to flash bootloader firmware to the ATTiny412 chip, can be found <a href="https://teddywarner.org/Projects/SerialUPDI/#jtag2updi-programming">**here**</a>.

I then modified Arduino IDE's example Blink code to use pin 4 as the output pin.

```cpp
int LED = 4;
// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED, HIGH);   // turn the LED on (HIGH is the voltage level)
  delay(1000);                       // wait for a second
  digitalWrite(LED, LOW);    // turn the LED off by making the voltage LOW
  delay(1000);                       // wait for a second
}
```

Here is a video of the board's LED blinking.

<center>
<video muted width="600" height="300" controls><source src="../../../pics/week6/workingATTiny.mp4" type="video/mp4" /></video>
</center>