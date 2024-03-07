# Bare Metal Programming

<pre><code class="language-cpp">#define LED_PIN 5

#define PORT_BASE 0x41004400UL
#define SYSCTRL_BASE 0x40000800UL

#define PORT_DIRSET (*((volatile unsigned long *)(PORT_BASE + 0x08)))
#define PORT_OUTTGL (*((volatile unsigned long *)(PORT_BASE + 0x1C)))
#define OSC8M (*((volatile unsigned long *)(SYSCTRL_BASE + 0x20)))

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
    OSC8M |= 0x1;
    PORT_DIRSET = (1 << LED_PIN); 
}

void loop() {
    PORT_OUTTGL = (1 << LED_PIN); 
    delay(); 
}

void delay() {
    for (volatile unsigned long i = 0; i < 2000000; i++) {
    }
}
</code></pre>