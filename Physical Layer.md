
# Layer 1: The Physical Layer ⚡

Alright, let's start at the absolute bottom.

Imagine we have two computers, and our goal is simple: get a piece of information from one to the other. Before we can get all philosophical about what "information" is, what it means, or why we're even here (lol), we face a much more basic problem. We need something that can _literally_ move stuff from here to there.

That's our first stop. This is **Layer 1: The Physical Layer**. It’s the collection of wires, cables, and radio waves that form the tangible, "dumb" foundation for all the intelligent stuff we're going to build on top of it.

### So, What Does It _Actually_ Do?

Now the question is, what is its job?

If you've read the layered architecture page, you'll know each layer has very clear responsibilities it needs to fulfill. The entire focus of a layer is to meet that responsibility, and it doesn't care how the layers above or below it work. That's up to them.

The Physical Layer has one, and only one, core job: to take the raw **bits** (the 1s and 0s) that the layer above it wants to send and convert them into some kind of physical signal that can be sent over a medium.

That's it. It’s a translator between the digital world and the physical world.

### Services for the Layer Above

Instead of just listing out our own ideas, let's look at the official source—the document that actually defines the OSI model. This tells us exactly what services the Physical Layer must offer to its only "customer": the Data Link Layer (Layer 2).

<img width="902" height="627" alt="Screenshot 2025-09-12 133658" src="https://github.com/user-attachments/assets/171451d8-170e-4d3b-9761-cad59c079e6a" />

Here’s a screenshot from the official standard document - [ISO/IEC 7498-1: The Basic Model](https://www.ecma-international.org/wp-content/uploads/s020269e.pdf "null")

It is very deliberate. It first establishes the layer's overall **Purpose** (its high-level mission) and then details the specific **Services** it must provide to the layer above. Let's break down those services:

-   **Physical Connections:** This is the most obvious one. The layer provides a path to send bits. It activates and deactivates this connection on behalf of the Data Link Layer. Think of it as opening and closing a channel for communication.
    
-   **Physical Service Data Units (PSDUs):** This is a fancy term for a simple idea. The Physical Layer doesn't send just one bit at a time. It's usually given a group of bits by the Data Link Layer and is responsible for sending that entire group. The service it provides is the transmission of this "unit" of bits.
    
-   **Sequencing:** The Physical Layer must ensure that if the Data Link Layer gives it a sequence of bits like `1-1-0-1`, it sends them in that exact order. It provides a reliable, sequential delivery of the bitstream.
    
-   **Fault Condition Notification:** What happens if the cable gets unplugged or there's a major electrical fault? The Physical Layer can detect this physical failure and has a responsibility to notify the Data Link Layer that something has gone wrong with the connection.
    
-   **Quality of Service (QoS) Parameters:** The Physical Layer can report on the "quality" of the connection. This could include the transmission rate (how fast it's sending bits), the error rate, and the transmission delay. This information allows the higher layers to make smarter decisions.
    

### The "Hands-On" Part: Sending a Real Character

Theory is great, but let's make this real. We're going to build a complete, end-to-end communication link in Tinkercad to send a meaningful piece of data—a single character—from a "Sender" to a "Receiver."

**The Goal:** To type a character (like 'A') on our Sender computer, transmit it bit-by-bit as high and low voltages, and see the exact same character appear on our Receiver computer.

**The Setup:**

To make this happen, we'll use the following components in our Tinkercad circuit:

-   **2 Arduino Unos:** One will act as our Sender, the other as our Receiver.
    
-   **1 Mini Breadboard:** This will help us create a clean junction point for our data line.
    
-   **1 Multimeter:** We'll set it to Voltmeter mode to measure the physical signal in real-time.
    

Here’s how to wire everything together, step-by-step:

1.  **Connect the Ground:** This is the most critical step. Run a wire from a `GND` pin on the Sender Arduino to a `GND` pin on the Receiver Arduino. This creates a common reference point for voltage. Without it, the Receiver can't reliably interpret the Sender's signals.
    
2.  **Connect the Data Line:** Run a wire from digital pin `8` on the Sender to a row on the breadboard. Then, run a second wire from the _same row_ on the breadboard to digital pin `8` on the Receiver. This wire is our physical medium.
    
3.  **Connect the Voltmeter:** Connect the voltmeter's positive (red) probe to the same breadboard row as the data line. Connect the negative (black) probe to the common ground wire.
    

Your final setup should look something like this. You can also view and run the completed project directly on [Tinkercad](https://www.tinkercad.com/things/9UM9vD7kOSU/editel?sharecode=kNEICPJylDNsixSnFyKQoW4q_0M0qtctTvmNHstr4z0 "null").

Now, let's get to the code that will bring this circuit to life.

#### The Sender Code (Device A)

```
// Sender Code
// Sends a character bit by bit with start + stop bits

const int dataPin = 8;
const int bitRateDelay = 1000; // 1 bit per second (slow for demo)

void setup() {
  pinMode(dataPin, OUTPUT);
  digitalWrite(dataPin, HIGH); // idle line = HIGH
  Serial.begin(9600);
  Serial.println("Enter a character to send:");
}

void loop() {
  if (Serial.available() > 0) {
    char charToSend = Serial.read();
    Serial.print("Sending character: ");
    Serial.println(charToSend);

    // --- Start bit (LOW) ---
    digitalWrite(dataPin, LOW);
    delay(bitRateDelay);

    // --- Send 8 data bits (LSB first, UART style) ---
    for (int i = 0; i < 8; i++) {
      int bit = bitRead(charToSend, i);
      digitalWrite(dataPin, bit ? HIGH : LOW);
      delay(bitRateDelay);
    }

    // --- Stop bit (HIGH) ---
    digitalWrite(dataPin, HIGH);
    delay(bitRateDelay);

    Serial.println("Character sent! Enter another:");
  }
}

```

This code improves on our basic idea by implementing a common protocol for sending data:

-   **Idle High:** The data line is kept at `HIGH` (5V) when nothing is being sent. This allows the receiver to detect when a new transmission begins.
    
-   **Start Bit:** To signal the start of a character, the sender drops the line to `LOW` for one bit-period. The receiver will be waiting for this specific transition.
    
-   **LSB First:** It sends the bits of the character from least-significant to most-significant, which is standard for serial communication.
    
-   **Stop Bit:** After the 8 data bits, it sends a `HIGH` signal for at least one bit-period to mark the end of the character and return the line to its idle state.
    

#### The Receiver Code (Device B)

```
// Receiver Code
// Waits for start bit, then reads 8 bits into a character

const int dataPin = 8;
const int bitRateDelay = 1000; // Must match sender

void setup() {
  pinMode(dataPin, INPUT);
  Serial.begin(9600);
  Serial.println("Receiver ready. Waiting for data...");
}

void loop() {
  // --- Wait for start bit (line goes LOW) ---
  while (digitalRead(dataPin) == HIGH);

  // Align to middle of first data bit
  delay(bitRateDelay + bitRateDelay / 2);

  byte receivedByte = 0;

  // --- Read 8 data bits (LSB first) ---
  for (int i = 0; i < 8; i++) {
    int bit = digitalRead(dataPin);
    if (bit == HIGH) {
      bitSet(receivedByte, i);
    }
    delay(bitRateDelay); // move to next bit
  }

  // --- Optional: wait for stop bit (HIGH) ---
  // Not strictly needed, since line idles HIGH.

  Serial.print("Character received: ");
  Serial.println((char)receivedByte);
}

```

The Receiver code is designed to perfectly synchronize with the Sender:

-   **Waits for the Start Bit:** The line `while (digitalRead(dataPin) == HIGH);` is a blocking loop. The code will do nothing until it sees the data line drop from its idle `HIGH` state to `LOW`. This is how it knows a character is coming.
    
-   **Timing and Synchronization:** Once the start bit is detected, it waits for one and a half bit-periods (`bitRateDelay + bitRateDelay / 2`). This is a clever trick to align the read operation to the _middle_ of the first data bit, making the reading more resilient to timing errors.
    
-   **Reassemblbles the Character:** It then reads the next 8 bits at the agreed-upon `bitRateDelay` interval, using `bitSet()` to build the character byte. Finally, it prints the received character to its own Serial Monitor.
