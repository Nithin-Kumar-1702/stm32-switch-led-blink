# Switch & LED Blinking with STM32G070RB (Low Power)

## Overview
This project demonstrates a register-level (CMSIS) implementation on the STM32G070RB NUCLEO board that cycles an LED through different blink rates using the onboard push button.

The application is designed to minimize power consumption by placing the CPU into Sleep Mode whenever it has no work to perform.

No HAL libraries or CubeMX-generated initialization code are used. All peripherals are configured through direct register access.

## Problem Statement
The onboard user button cycles the LED through the following states:

| Button Press | LED Behaviour |
|---|---|
| 1st | Blink at 0.5 Hz |
| 2nd | Blink at 1 Hz |
| 3rd | Blink at 2 Hz |
| 4th | LED Off |
| 5th | Repeat from 0.5 Hz |

The implementation must use the STM32's low-power features to reduce CPU power consumption.

## Hardware

### Target Board
- STM32 NUCLEO-G070RB
- MCU: STM32G070RBT6 (Cortex-M0+)

### Pin Assignment
| Signal | Pin | Description |
|---|---|---|
| LED | PA5 | Onboard User LED (LD4) |
| Switch | PC13 | Onboard User Button (B1), idle HIGH, active LOW |

## Design Approach

### Register-Level Programming
- Pure CMSIS implementation
- No HAL drivers
- No CubeMX-generated initialization
- Direct configuration of: RCC, GPIO, EXTI, TIM3, SysTick, NVIC

### Button Handling
The push button is handled entirely using EXTI Line 13 configured for falling-edge interrupts.

There is no polling, allowing the CPU to remain asleep until a button press occurs.

### Software Debouncing
Switch debouncing is implemented using a free-running SysTick millisecond counter.

- Debounce window: 50 ms
- Any interrupt occurring within 50 ms of the previous accepted press is ignored.
- No blocking delays are used.

### State Machine
The firmware maintains a four-state finite state machine.

| State | LED Behaviour |
|---|---|
| 0 | LED Off |
| 1 | 0.5 Hz |
| 2 | 1 Hz |
| 3 | 2 Hz |

Each valid button press advances to the next state.

```
0 → 1 → 2 → 3 → 0 → ...
```

### LED Timing
Blink timing is generated using TIM3.

Configuration:
- Timer clock: 16 MHz
- Prescaler: 15999
- Timer tick: 1 ms

Auto-reload values:

| Blink Rate | ARR Value | Toggle Interval |
|---|---|---|
| 0.5 Hz | 999 | 1000 ms |
| 1 Hz | 499 | 500 ms |
| 2 Hz | 249 | 250 ms |

The LED is toggled inside the TIM3 Update Interrupt. No software delay loops are used.

## Low-Power Operation
The main loop consists solely of:

```c
while (1)
{
    __WFI();
}
```

The CPU enters Sleep Mode between interrupts and wakes only for:
- Button press (EXTI13)
- Timer interrupt (TIM3 Update)

This significantly reduces CPU activity while keeping all required peripherals operational.

### Why Sleep Mode?
- CPU clock stopped
- Peripheral clocks remain active
- TIM3 continues running
- EXTI remains functional
- State variables are preserved

### Why Not Stop Mode?
Stop mode disables peripheral clocks. Since TIM3 would stop counting, LED blinking would also stop unless the design were migrated to an LPTIM capable of operating during Stop mode.

### Why Not Standby Mode?
Standby mode powers down almost the entire device. On wake-up: peripheral configuration is lost, RAM contents are not preserved, and state variables must be restored manually. This makes it unnecessarily complex for the assignment requirements.

## Assumptions

### System Clock
Default 16 MHz internal HSI oscillator. No PLL configuration. No external crystal.

### Debounce Time
A debounce period of 50 ms is used. This is a typical value for mechanical tactile switches and can be adjusted if necessary.

### Blink Frequency
Blink frequency refers to the number of complete ON/OFF cycles per second.

| Blink Rate | LED Toggle Interval |
|---|---|
| 0.5 Hz | 1000 ms |
| 1 Hz | 500 ms |
| 2 Hz | 250 ms |

### LED Off State
When entering the OFF state, TIM3 is stopped and the LED output is explicitly cleared. This guarantees the LED is always OFF rather than remaining in its previous state.

This same state-0 logic is invoked once at boot, immediately after peripheral setup completes (`set_blink_state(current_state)` with `current_state = 0`). This guarantees the LED starts genuinely OFF on power-up, rather than relying on GPIO/timer reset defaults, which this design does not depend on.

### Low-Power Scope
Only Sleep Mode is implemented. Supporting Stop or Standby modes would require a different timer architecture (such as LPTIM), which is beyond the scope of this assignment.

## Hardware Validation
The firmware has been:
- Desk-checked
- Register values verified
- State transitions reviewed
- Timing calculations validated

At the time of writing, physical hardware testing was pending due to board availability.

## Toolchain
- STM32CubeIDE 2.1.1
- CMSIS (stm32g0xx.h, core_cm0plus.h)
- Register-level programming
- No HAL
- No CubeMX initialization code

## Project Features
- Register-level STM32 programming
- Interrupt-driven button handling
- Software debounce
- Timer-based LED blinking
- Finite State Machine implementation
- Low-power CPU operation using `__WFI()`
- No polling
- No busy-wait delays
- No HAL dependencies

## Repository Structure
```
.
├── Src/
│   ├── main.c
│   ├── syscalls.c
│   └── sysmem.c
├── Startup/
│   └── startup_stm32g070rbtx.s
├── .cproject
├── .project
├── .gitignore
├── STM32G070RBTX_FLASH.ld
└── README.md
```

## Project Organization
- `Src/` – Contains the application source code and system support files.
- `Startup/` – Contains the startup assembly file responsible for MCU initialization and vector table setup.
- `STM32G070RBTX_FLASH.ld` – Linker script defining the memory layout of the STM32G070RB.
- `.project` and `.cproject` – STM32CubeIDE project configuration files.
- `README.md` – Project documentation describing the design, implementation, and assumptions.

## License
This project was developed as part of an embedded systems assignment and is intended for educational purposes.
