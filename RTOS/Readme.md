Source: https://youtube.com/playlist?list=PLEBQazB0HUyQ4hAPU1cJED6t3DU0h34bz&feature=shared

The video "Introduction to RTOS Part 1 - What is a Real-Time Operating System (RTOS)?" from Digi-Key Electronics introduces the concept of a Real-Time Operating System (RTOS) and its applications, particularly focusing on FreeRTOS.

Here's a breakdown of the video's content:

*   **What an Operating System (OS) Does**
    An operating system is a software component on a computer or microcontroller responsible for several key functions:
    *   **Scheduling tasks and applications**: It allocates "slices of time" to various processes, making them appear to run concurrently.
    *   **Managing virtual resources**: It allows applications to access resources like files, libraries, and folders.
    *   **Providing device drivers**: These drivers enable the system to interact with external hardware, such as reading/writing from disks, responding to input, or drawing graphics.

*   **General Purpose Operating Systems (GPOS) vs. Real-Time Operating Systems (RTOS)**
    *   **GPOS (e.g., Windows, macOS, Linux, iOS, Android)**: These are designed primarily for human interaction. Their schedulers prioritize user experience, meaning some timing deadlines might be missed or delayed if not noticeable to a human. GPOS schedulers are often **non-deterministic**, meaning it's impossible to precisely predict when a task will execute or for how long.
    *   **RTOS**: Unlike GPOS, an RTOS is specifically designed to **guarantee meeting timing deadlines** for tasks. This is crucial for applications like medical devices or engine controllers, where missing a deadline (e.g., firing a spark plug) could be catastrophic. While some RTOSes offer high-level device drivers, they often focus on microcontroller-specific elements like Wi-Fi/Bluetooth stacks or simple LCD drivers. Bare-bones RTOSes might only provide a scheduler, requiring users to implement their own file systems and device drivers.

*   **Why use an RTOS?**
    The primary reasons to use an RTOS include:
    *   **Guaranteed timing deadlines**: Essential for critical applications where precise timing is non-negotiable.
    *   **Concurrent task execution**: An RTOS allows multiple tasks to run "at the same time" (through time-slicing on single-core processors), which can simplify complex projects and improve responsiveness.
    *   **Resource management**: It helps manage CPU time and memory, especially on more powerful microcontrollers.
    *   **Complex project organization**: For team projects, an RTOS can help divide features into individual, concurrently running tasks among team members.

*   **When NOT to use an RTOS (and alternatives)**
    The video also discusses scenarios where an RTOS might not be the best fit:
    *   **Super Loop Architecture**: For simple microcontroller projects with a handful of tasks, a "super loop" (setup functions followed by tasks executing in a round-robin fashion within a `while(forever)` loop) is often sufficient. It's easy to implement, has low overhead, saves CPU cycles and memory, and is easier to debug.
    *   **Interrupt Service Routines (ISRs)**: If only one or two tasks require strict timing deadlines (especially under 1 millisecond), using ISRs might be a better and more precise solution. For intervals less than a few hundred nanoseconds, a very fast processor or custom hardware like an FPGA might be needed.
    *   **Resource-constrained microcontrollers**: For simple 8 or 16-bit microcontrollers with limited RAM (e.g., 2 KB), the memory and CPU overhead of an RTOS scheduler can leave few resources for the actual program. In such cases, a super loop is generally preferable.

*   **Key RTOS Concepts and Terminology**
    The video introduces important terms:
    *   **Tasks/Threads**: A "task" is a piece of work, while a "thread" is a unit of CPU utilization with its own program counter and memory. In FreeRTOS, the term "task" is used more like a thread.
    *   **Prioritization**: Tasks can be assigned different priorities, allowing more critical tasks to execute sooner or receive more CPU time.
    *   **Inter-task communication**: Methods for tasks to exchange information.
    *   **Memory Management**: How the OS handles memory allocation for tasks and processes.
    *   **Processes**: An instance of a running computer program. A process usually has one or more threads. General-purpose OSes can run many processes, while an RTOS often handles only one.
    *   **Interrupts**: Interrupts still function within an RTOS. If an interrupt has a higher priority than any task, it will pause all task execution, run its Interrupt Service Routine (ISR), and then return control to where it left off.

*   **FreeRTOS and ESP32**
    The series will use **FreeRTOS** because it is the most popular RTOS for IoT devices, is free and open-source, and is maintained by Amazon. The **ESP32 microcontroller** is chosen for demonstrations due to its power, features, affordability, and the fact that it runs a modified version of FreeRTOS out of the box, simplifying setup within the Arduino environment. Using Arduino also creates a level playing field for many embedded programmers. The video promises to cover concepts like prioritization and resource management with FreeRTOS on the ESP32, including creating tasks to blink an LED in the next video.

In essence, the video explains that while general-purpose operating systems aim for a smooth user experience, RTOSes are built to ensure critical tasks meet their deadlines, making them indispensable for systems where timing failures could have severe consequences. It also highlights that for simpler applications, a "super loop" or interrupts might be more appropriate than an RTOS.

---

This video, "Introduction to RTOS Part 3 - Task Scheduling," from Digi-Key Electronics, builds upon the previous discussion of FreeRTOS by explaining the **fundamental mechanisms of task scheduling, preemption, and task states** within a Real-Time Operating System, particularly in the context of the ESP32 and Arduino IDE.

Here's a detailed explanation of the video's content:

### 1. How the RTOS Scheduler Works

*   **Multi-threading vs. Single-Core Reality**: While a multi-threaded program appears to run multiple `while forever` loops simultaneously, a single-core processor must **divide its time among these tasks**.
*   **Time Slicing and Ticks**: Most RTOSs, including FreeRTOS on the ESP32, use **time slicing**. A hardware timer interrupts the processor at regular intervals, typically **one millisecond**, known as a "tick".
*   **Scheduler's Role**: At every tick, the operating system's scheduler is called. Its job is to **determine which task should run next** based on its priority and state.
*   **Priority-Based Scheduling**: Tasks are assigned priority levels (e.g., 0 for lowest, up to 24 in FreeRTOS on ESP32). The scheduler always chooses the **highest priority task that is ready to run**.
*   **Pre-emptive Scheduling**: If a higher-priority task becomes ready, the scheduler will **interrupt a currently running lower-priority task** to allow the higher-priority task to execute. This is known as pre-emptive scheduling, where CPU time is taken from one task to run another.
*   **Round-Robin for Equal Priority**: If multiple tasks of the same highest priority are ready, the scheduler executes them **in turn, in a round-robin fashion**.

### 2. Task States

The FreeRTOS scheduler keeps track of each task's state:

*   **Ready State**: When a task is created, it automatically enters the **ready state**. In this state, the task is telling the scheduler it's prepared to run at any time, but it will only be chosen if no other higher-priority tasks are waiting. If not chosen, it remains in the ready state.
*   **Running State**: If a task is selected by the scheduler to run, it enters the **running state** and uses the processor. On a single-core system, only one task can be in the running state at any given time.
*   **Blocked State**: A task can enter the **blocked state** by calling an API function like `vTaskDelay()` (which waits for a specified number of ticks) or waiting for a queue or semaphore. Tasks in this state do not run and cannot be selected by the scheduler. They wait for an unblocking event (e.g., delay expiring, semaphore release), after which they return to the ready state.
*   **Suspended State**: A task can be explicitly put into the **suspended state** using the `vTaskSuspend()` API function (from within itself or another task). Like the blocked state, a suspended task cannot be selected to run. However, it can only return to the ready state via an explicit call to `vTaskResume()`. This is useful for temporarily "sleeping" tasks without relying on timers.

### 3. Context Switching

*   When the scheduler switches from one task to another, it performs a **context switch**.
*   This involves **remembering the exact point where the current task left off**, including all its working variables, values in RAM, and CPU registers. This complete snapshot of a task's working environment is called its **context**.
*   The scheduler then **restores the context of the next task** to be run, allowing it to resume precisely where it previously stopped.
*   The **stack allocated for the task** is crucial for storing much of this context.

### 4. Hardware Interrupts

*   Hardware interrupts (like those for timer overflows or pin changes) always have a **higher priority than any software task** running, unless explicitly disabled in code.
*   When an Interrupt Service Routine (ISR) is finished, execution returns to whichever task was running before the interrupt occurred.
*   The video advises keeping ISRs as short as possible.

### 5. Multi-core Systems (ESP32)

*   While multi-core systems (like the ESP32) *can* allow a scheduler to allocate tasks to different cores for simultaneous execution (e.g., Task B and C running at the exact same time), the series continues to **limit examples to using only one core** for simplicity.

### 6. Practical Demonstration of Task Preemption

The video demonstrates these concepts with an Arduino example on the ESP32:

*   **Setup**: The example uses a slow serial baud rate (300) to visually observe the preemption in real-time. It also highlights that the Arduino `setup()` and `loop()` functions run within their own FreeRTOS task, with priority one on core one.
*   **Task 1 (Lower Priority - 1)**: This task prints a "pirate talk" string character by character to the serial terminal. After printing, it uses `vTaskDelay()` to enter the blocked state for one second.
*   **Task 2 (Higher Priority - 2)**: This task simply prints a single asterisk character every 100 milliseconds.
*   **Control Task (Setup/Loop Task)**: This task suspends and resumes Task 2 (the asterisk printer) every two seconds. Later, it completely **deletes Task 1** using `vTaskDelete()`.
    *   **Crucial Safety Tip**: When deleting a task, it's highly recommended to check if the task handle is not null and then immediately set the handle to null after deletion to prevent memory corruption or crashes if `vTaskDelete()` is called on a non-existent task.
*   **Observation**:
    *   The serial output shows the "pirate talk" sentence being printed.
    *   When Task 2 is active, asterisks appear in the middle of the sentence, demonstrating **pre-emptive scheduling** where the higher-priority Task 2 interrupts Task 1.
    *   The asterisks start and stop as Task 2 is suspended and resumed.
    *   After Task 1 is deleted, only the asterisk-printing Task 2 continues to run.

### Challenge and Next Steps

The video concludes with a challenge for the user to create a program where one task blinks an LED, and a second task listens for serial input to dynamically update the LED's blinking delay time, creating a simple user interface. The next video in the series will focus on memory allocation within FreeRTOS, covering stack, heap, and static memory.

**In essence, the RTOS scheduler acts like a meticulous conductor of an orchestra where each musician is a task.** The conductor's job is to ensure that the most important musicians (highest priority tasks) play precisely when needed, even if it means interrupting another musician's solo. It also keeps track of where each musician left off (context switching) and gives them breaks (blocked state) or tells them to completely stop until explicitly called again (suspended state), all while relying on a metronome (the tick timer) to keep perfect timing.
