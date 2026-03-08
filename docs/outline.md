This project is essentially a "Speedrun of Computing History." You are moving from the 2020s (Distributed Cloud) back to the 1980s (Bare Metal) to see how the foundations were laid.

In your day job, you manage **clusters**. In this project, you manage **clock cycles**.

---

## Project Architecture: The "Zero-Stack" Chat

The architecture follows a "Bottom-Up" approach. You start by manipulating pixels, then move to bits on a wire, and finally, you hijack the CPU’s attention via interrupts.

### Phase 1: The Virtual Lab (Infrastructure)

Since you’re a Docker pro, we use it to create a reproducible hardware environment.

* **Node A (Server):** A DOSBox-X container listening on a TCP port.
* **Node B (Client):** A DOSBox-X container connecting to Node A.
* **The "Wire":** A virtual null-modem cable. In `dosbox.conf`, you map `COM1` to a TCP socket.
* **The Compiler:** Open Watcom C16. It’s the industry standard for 16-bit DOS development.

### Phase 2: Direct Memory Access (The Screen)

Forget `printf`. To understand the "CS Degree" concept of **Memory Mapped I/O**, you will write directly to video RAM.

* **The Goal:** Write a function `draw_char(x, y, char, color)`.
* **The Tech:** In DOS Real Mode, the screen is just an array of bytes starting at memory address `0xB800:0000`.
* **The Lesson:** Every even byte is the character; every odd byte is the color attribute.

### Phase 3: Talking to the Silicon (UART Polling)

This is where you replace "Azure Event Hubs" with a hardware chip called the **UART** (Universal Asynchronous Receiver-Transmitter).

* **Initialization:** You must "program" the chip by sending bytes to specific **I/O Ports** (like `0x3F8`). You'll set the Baud Rate, Parity, and Stop Bits.
* **The Polling Loop:** 1.  Check the **Line Status Register** (LSR).
2.  Is the "Data Ready" bit (Bit 0) set to 1?
3.  If yes, read the byte from the **Receive Buffer Register**.
* **The Lesson:** You'll experience "Busy Waiting." Your CPU will be pegged at 100% just looking at a single bit over and over.

### Phase 4: Flow Control (The Circular Buffer)

In your cloud job, you have infinite scaling. In DOS, if a burst of data comes in while your CPU is busy drawing to the screen, the UART’s tiny 1-byte buffer will overflow.

* **The Goal:** Implement a **Circular (Ring) Buffer**.
* **The Tech:** Create an array with `Head` and `Tail` pointers. As bytes come in, put them in the array. As the UI loop is ready, pull them out.
* **The Lesson:** This is your first taste of **Manual Backpressure**.

### Phase 5: The Final Boss (Hardware Interrupts)

This is the "CS Degree" holy grail. You will move from **Polling** to **Event-Driven Architecture** at the hardware level.

* **The Goal:** The CPU should only care about the Serial Port when a byte actually arrives.
* **The Tech:** 1.  Write an **Interrupt Service Routine (ISR)**—a "naked" C function.
2.  Modify the **Interrupt Vector Table (IVT)** at the base of RAM to point IRQ4 to your function.
3.  When a bit hits the wire, the CPU stops your `main()` loop, runs your ISR, and then jumps back.
* **The Lesson:** This is how `async` works under the hood.

---

## The Detailed Project Roadmap

| Milestone | Task | Cloud Equivalent | "Aha!" Moment |
| --- | --- | --- | --- |
| **1. The Lab** | Docker-Compose with two DOSBox-X nodes linked via TCP. | Setting up a VNET / VPC. | "Networking is just a file descriptor." |
| **2. Video** | Write "Hello" by poking `0xB800` memory addresses. | Front-end Rendering. | "The screen is just a specialized RAM bank." |
| **3. Talking** | Send a single 'A' to the other node using `outp(0x3F8, 'A')`. | Sending a POST request. | "I'm physically moving electrons now." |
| **4. Buffering** | Implement a 256-byte Ring Buffer for incoming text. | Kafka / RabbitMQ. | "If I don't catch this byte, it's gone forever." |
| **5. Interrupts** | Hook IRQ4 to trigger your code on data receipt. | Webhooks / Azure Functions. | "The hardware is calling *me*." |

---

## Why this is "Fun" for a Developer

You've spent 3 years in a world where a "Service Unavailable" error is a mystery solved by looking at logs. In this project, a "Service Unavailable" error means **you** forgot to clear a bit in the Interrupt Controller, and you have to reboot the whole "datacenter" (your VM). It is incredibly satisfying to see two separate windows chatting using code you wrote from the transistor up.

