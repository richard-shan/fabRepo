# Bare Metal Programming

Our local instructor, Mr Dubick, required each student to program either one of the SAMD11C or the ATTINY412 chip with a low level, bare metal program. I decided to use the SAMD11C chip as that was the chip I worked with the most this week and was the most familiar with.

During the many hours of my previous attempt at sending code <i> through </i> the RP2040, I had corresponded with <a href="https://fabacademy.org/2020/labs/leon/students/adrian-torres/">**Adrian Torres'**</a>. During our conversation, he directed me to a sample piece of code on the Fab Academy website that I could use to test the board.

<pre><code class="language-cpp">// hello.D11C.blink.ino
//
// SAMD11C LED blink hello-world
//
// Neil Gershenfeld 11/29/19
//
// This work may be reproduced, modified, distributed,
// performed, and displayed for any purpose. Copyright is
// retained and must be preserved. The work is provided
// as is; no warranty is provided, and users accept all 
// liability.
//

#define LED (1 << 5) // define LED ping

void setup() {
   SYSCTRL->OSC8M.bit.PRESC = 0; // set OSC8M clock prescaler to 1
   REG_PORT_DIR0 |= LED; // set LED pin to output
   }

void loop() {
   REG_PORT_OUT0 |= LED; // turn on LED
   delay(100); // delay
   REG_PORT_OUT0 &= ~LED; // turn off LED
   delay(100); // delay
   }
</code></pre>

However, I didn't really understand the code. After doing some research online, I realized that although the code worked, it still operated at a decently high level in its usage of the delay() function. I decided to try to code at an even lower level. I would find out that this sample code used a massive amount of memory when compared to lower level programs that do the same function.

<pre><code class="language-none">Sketch uses <span style="color:red"><b>9760 bytes (79%)</b></span> of program storage space. Maximum is 12288 bytes.</code></pre>

As I usually program in high level C-based programming languages, I had to do some research online before writing my own low level program (thanks Stack Overflow!). I came up with the following program, which I sent to many of my fellow students.

<pre><code class="language-cpp">#define LED_PIN 5 // Sets the LED pin number to 5

// Define base addresses and offsets as constants
#define PORT_BASE 0x41004400UL
#define SYSCTRL_BASE 0x40000800UL

// Define the register addresses directly
#define PORT_DIRSET (*((volatile unsigned long *)(PORT_BASE + 0x08)))
#define PORT_OUTTGL (*((volatile unsigned long *)(PORT_BASE + 0x1C)))
#define OSC8M (*((volatile unsigned long *)(SYSCTRL_BASE + 0x20)))

// Function prototypes 
void setup();
void loop();
void delay();

int main() {
    setup();
    while (1) {
        loop();
    }
}

void setup() {
    OSC8M |= 0x1; // Set OSC8M clock prescaler to 1
    PORT_DIRSET = (1 << LED_PIN); // Set LED pin to output
}

void loop() {
    PORT_OUTTGL = (1 << LED_PIN); // Toggle LED state
    delay(); // Delays
}

void delay() {
    for (volatile unsigned long i = 0; i < 2000000; i++) {
        // Does nothing but still functions as a delay
    }
}</code></pre>

Most of the code is explained in the comments, but I was especially interested in how to set the time of the delay. 2,000,000 was a suggested value to use for a 8MHz clock to delay a second. 8MHz means that the clock is running at 8 million cycles per second. Unfortunately, 2000000 is a rough measurement and each chip is a little bit different from any other chip. In the future, I would manually calibrate the chip's individual clock.

<pre><code class="language-none">Sketch uses <span style="color:green"><b>3628 bytes (29%)</b></span> of program storage space. Maximum is 12288 bytes.</code></pre>

After some more time optimizing, I managed to reduce the memory usage of the program by 12 bytes. This is the final program that I would end up using.

<pre><code class="language-cpp">// Define the base memory address
#define PORT_BASE 0x41004400

// Define access to the Data Direction Set Register (DIRSET)
#define PORT_DIRSET (*((volatile unsigned long *)(PORT_BASE + 0x08)))

// Define access to the Output Toggle Register (OUTTGL)
#define PORT_OUTTGL (*((volatile unsigned long *)(PORT_BASE + 0x1C)))

// Define the base memory address for the System Control (SYSCTRL) registers
#define SYSCTRL_BASE 0x40000800

// Define access to the Register of the 8MHz internal oscillator (OSC8M)
#define OSC8M (*((volatile unsigned long *)(SYSCTRL_BASE + 0x20)))

void setup() {
  OSC8M |= 0x1;   // Enable the internal 8MHz oscillator by setting the first bit of the OSC8M register.

  PORT_DIRSET = (1 << 5);   // Set the 5th pin as an output 
}

void loop() {
  PORT_OUTTGL = (1 << 5);   // Toggle the state of the 5th pin 
  for (volatile unsigned long i = 0; i < 2000000; i++); // Iterates 2,000,000 times to create a rough delay.
}

int main() {
  setup();
  while (1) {
    loop();
  }
}</code></pre>

All 3 of these programs blink the Hello SAMD11C Board's LED which is located on Pin 5. This is done through toggling the power coming out of Pin 5. In addition to taking up less memory, programs 2 and 3 also upload to the chip faster than program 1. The main memory-saving difference between program 1 and program 3 is that program 1 uses the delay() function, while program 3 creates my own low-level way to wait for an amount of time.