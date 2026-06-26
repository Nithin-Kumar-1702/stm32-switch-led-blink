# Sensor Buffer Simulation (Producer-Consumer in C)

A simple C program that simulates a sensor generating random data every second (producer) and a separate process that periodically checks, prints, and clears the latest data from a shared buffer (consumer).

## Problem Statement

1. A timer-driven producer triggers every **1 second**, generating a random number (0–5) of random bytes and appending them to a globally accessible buffer.
2. A separate consumer wakes up every **10 seconds**, checks if the buffer holds at least 50 bytes, and if so, prints only the **latest 50 bytes** (in hex) and removes them from the buffer. If fewer than 50 bytes are available, it does nothing and waits for the next cycle.

## How It Works

- `rand_byte_count()` — returns a random count (0–5) representing how many bytes the sensor produces this tick.
- `rand_byte_data()` — returns a single random byte value (0–255).
- `producer_tick()` — called once per simulated second; generates 0–5 random bytes and appends them to the buffer.
- `consumer_tick()` — called once per simulated 10 seconds; if the buffer has ≥50 bytes, prints the latest 50 in hex and removes them, preserving any older leftover bytes.
- `run_simulation(int total_sec)` — drives the simulation loop, calling `producer_tick()` every iteration and `consumer_tick()` every 10th iteration.

## Build & Run

### Locally (GCC)
```bash
gcc main.c -o sensor_sim
./sensor_sim
```

### Online Compiler
This program uses only standard C libraries (`stdio.h`, `stdlib.h`, `time.h`) and has been tested and verified to compile and run successfully on [Programiz Online C Compiler](https://www.programiz.com/c-programming/online-compiler/).

## Sample Output

```
t= 1 sec Added 2 Bytes
t= 2 sec Added 2 Bytes
t= 3 sec Added 5 Bytes
...
t= 20 sec Added 54 Bytes
Printing 50 Bytes in Hex:
14 a6 39 33 c7 65 99 a8 42 71 97 1 d 5 81 c1 59 fe 90 91 25 e1 21 6e 15 3 d9 87 af e7 3b d9 da 1f 57 2f c7 9e 5a 19 b0 c3 39 ae 68 e9 a 52 3c 5d
4
t= 21 sec Added 5 Bytes
...
```

At t=20, 54 bytes had accumulated. The consumer printed the latest 50 (in hex) and left 4 bytes behind, which carried forward into the next cycle.

## Assumptions

1. **Buffer size:** A fixed-size global array of 200 bytes (`BUFFER_MAX`) is used to store incoming sensor data. This is comfortably larger than the maximum that could realistically accumulate between consumer cycles (worst case: 5 bytes/sec × 10 sec = 50 bytes per cycle), giving headroom in case the consumer is delayed or skipped for a cycle. If the buffer ever fills completely, the producer simply stops adding new bytes until space is freed by the consumer (no overflow/wraparound is implemented).

2. **"Latest 50 bytes" definition:** The most recently added 50 bytes are treated as the "latest" — i.e., the last 50 entries in the buffer at the time the consumer runs, regardless of how many seconds' worth of data they span.

3. **Leftover byte handling:** When the consumer prints and removes the latest 50 bytes, any older bytes that were already in the buffer before those 50 are preserved (not discarded). They remain at the front of the buffer and are carried forward into the next cycle, where new bytes from the producer continue to be appended after them.

4. **Timing simulation:** Since this program targets simple online compilers (e.g., Programiz) without real multithreading or hardware timers, time is simulated using a single loop where each iteration represents one elapsed second. The producer runs on every iteration (simulating a 1-second timer), and the consumer's check runs every 10th iteration (simulating a 10-second timer). This approximates two independent periodic timers within a single-threaded program.

5. **Randomness:** `srand(time(NULL))` is called exactly once, at the very start of `main()`, to seed the random number generator. Seeding more than once (e.g., inside a function called every second) would produce repeated/non-random sequences within the same second, since `time(NULL)` has 1-second resolution.

6. **No data persistence:** All data lives in memory for the duration of the program's run; no file storage is used, since the task is about simulating a real-time producer-consumer pattern, not long-term storage.

## Tools Used

- Language: C (C99)
- Development & testing: VS Code (local) and Programiz Online Compiler (verification)
