# Experiment 13 — Eight-LED Sequential Lighting System Using AT89C51

> **Project name:** LED Motion Engine  
> **Course:** 19EC408 — Microprocessors and Microcontrollers Laboratory  
> **Controller:** AT89C51  
> **Language:** 8051 Assembly (Keil A51)  
> **Tools:** Keil µVision and Proteus

## Aim

To design and simulate an eight-LED sequential lighting system using the AT89C51 microcontroller. The system provides forward, reverse, and bounce lighting patterns selected using two input switches.

## Components Required

| Component | Specification | Quantity |
|---|---|---:|
| Microcontroller | AT89C51 | 1 |
| LEDs | Standard indicator LEDs | 8 |
| Current-limiting resistors | 330 Ω (verify current for the selected LED) | 8 |
| Crystal | 11.0592 MHz | 1 |
| Crystal capacitors | 33 pF (typical starting value) | 2 |
| Mode switches | SPST switches / push buttons | 2 |
| Regulated supply | +5 V DC | 1 |
| Development software | Keil µVision with A51 assembler | — |
| Simulation software | Proteus | — |

## Pin Connections

| AT89C51 pin | Connection |
|---|---|
| P2.0 (pin 21) | LED1 |
| P2.1 (pin 22) | LED2 |
| P2.2 (pin 23) | LED3 |
| P2.3 (pin 24) | LED4 |
| P2.4 (pin 25) | LED5 |
| P2.5 (pin 26) | LED6 |
| P2.6 (pin 27) | LED7 |
| P2.7 (pin 28) | LED8 |
| P1.0 (pin 1) | Mode input bit 0 |
| P1.1 (pin 2) | Mode input bit 1 |
| XTAL1 / XTAL2 (pins 19 / 18) | 11.0592 MHz crystal circuit |
| RST (pin 9) | Reset circuit |
| VCC (pin 40) | +5 V |
| GND (pin 20) | Ground |

**Switch wiring:** This program uses active-LOW switch inputs. Connect each switch between its port pin and GND. The 8051 quasi-bidirectional port's internal pull-up keeps an unpressed input HIGH; pressing a switch reads LOW. Do not add a pull-down resistor to this active-LOW arrangement.

**LED wiring:** The program assumes active-HIGH LEDs: a port bit of 1 turns its LED ON. Confirm the LED orientation and current path in Proteus. If your circuit is active-LOW, the output patterns must be inverted.

## Mode Selection

| P1.1 | P1.0 | Mode | Sequence |
|---:|---:|---|---|
| 1 | 1 | Forward | LED1 → LED2 → … → LED8 |
| 1 | 0 | Reverse | LED8 → LED7 → … → LED1 |
| 0 | 1 | Bounce | LED1 → … → LED8 → … → LED2, then repeat |
| 0 | 0 | Reserved | All LEDs OFF |

The mode is checked after each complete pattern cycle. Therefore, a switch change takes effect when the current cycle finishes.

## Working Principle

Port 2 carries an 8-bit pattern. In forward mode, the active bit is rotated left; in reverse mode, it is rotated right. Bounce mode reads a sequence of byte values from a lookup table using the 8051 MOVC instruction. A software delay makes each LED state visible.

For active-HIGH LED wiring, the forward sequence is:

| LED | P2 value | Binary |
|---:|---:|---|
| 1 | 01H | 00000001 |
| 2 | 02H | 00000010 |
| 3 | 04H | 00000100 |
| 4 | 08H | 00001000 |
| 5 | 10H | 00010000 |
| 6 | 20H | 00100000 |
| 7 | 40H | 01000000 |
| 8 | 80H | 10000000 |

## Algorithm

1. Initialise the stack pointer, switch input port and LED output port.
2. Read P1.1 and P1.0.
3. Select forward, reverse, bounce or reserved mode.
4. Output the next LED pattern to Port 2.
5. Execute the software delay.
6. Repeat the selected sequence.
7. Check the mode switches again after the sequence completes.

## 8051 Assembly Program

The source file is available at [src/exp13/led_motion.a51](./src/exp13/led_motion.a51).

~~~asm
            ORG 0000H
            LJMP START

START:
            MOV SP, #5FH
            MOV P1, #0FFH
            MOV P2, #00H

MAIN:
            JB  P1.1, CHECK_P10
            JB  P1.0, BOUNCE_MODE
            SJMP IDLE_MODE

CHECK_P10:
            JB  P1.0, FORWARD_MODE
            SJMP REVERSE_MODE

FORWARD_MODE:
            MOV A, #01H
            MOV R5, #08H

FORWARD_LOOP:
            MOV P2, A
            ACALL DELAY
            RL  A
            DJNZ R5, FORWARD_LOOP
            SJMP MAIN

REVERSE_MODE:
            MOV A, #80H
            MOV R5, #08H

REVERSE_LOOP:
            MOV P2, A
            ACALL DELAY
            RR  A
            DJNZ R5, REVERSE_LOOP
            SJMP MAIN

BOUNCE_MODE:
            MOV DPTR, #BOUNCE_TABLE
            MOV R0, #00H
            MOV R5, #0EH

BOUNCE_LOOP:
            MOV A, R0
            MOVC A, @A+DPTR
            MOV P2, A
            ACALL DELAY
            INC R0
            DJNZ R5, BOUNCE_LOOP
            SJMP MAIN

IDLE_MODE:
            MOV P2, #00H
            SJMP MAIN

DELAY:
            MOV R6, #0FFH
DELAY_OUTER:
            MOV R7, #0FFH
DELAY_INNER:
            DJNZ R7, DELAY_INNER
            DJNZ R6, DELAY_OUTER
            RET

BOUNCE_TABLE:
            DB 01H, 02H, 04H, 08H, 10H, 20H, 40H, 80H
            DB 40H, 20H, 10H, 08H, 04H, 02H

            END
~~~

The delay is an instruction-loop delay, not a calibrated millisecond timer. Its duration depends on the oscillator frequency and 8051 machine-cycle timing. Adjust the loop constants after observing the simulation.

## Expected Output

These are the expected port patterns for active-HIGH LED wiring. They describe the intended behaviour; they are not screenshots or proof of a completed Proteus run.

**Mode 1 — Forward (P1.1:P1.0 = 11)**

~~~text
01H → 02H → 04H → 08H → 10H → 20H → 40H → 80H → repeat
~~~

LED sequence: LED1 → LED2 → LED3 → LED4 → LED5 → LED6 → LED7 → LED8.

**Mode 2 — Reverse (P1.1:P1.0 = 10)**

~~~text
80H → 40H → 20H → 10H → 08H → 04H → 02H → 01H → repeat
~~~

LED sequence: LED8 → LED7 → LED6 → LED5 → LED4 → LED3 → LED2 → LED1.

**Mode 3 — Bounce (P1.1:P1.0 = 01)**

~~~text
01H → 02H → 04H → 08H → 10H → 20H → 40H → 80H → 40H → 20H → 10H → 08H → 04H → 02H → repeat
~~~

LED sequence: LED1 → LED2 → … → LED8 → LED7 → … → LED2 → repeat.

**Reserved mode (P1.1:P1.0 = 00):** P2 is set to 00H, so all active-HIGH LEDs are OFF.

## Keil µVision Build Procedure

1. Create a project and select AT89C51.
2. Add src/exp13/led_motion.a51 to the project as an A51 assembly source.
3. Enable HEX file generation in Options for Target → Output.
4. Build the target and check the build output for assembler errors.
5. Locate the generated HEX file in the project output directory.

## Proteus Simulation Procedure

1. Place the AT89C51, eight LEDs, eight series resistors, crystal circuit, reset circuit and two switches.
2. Connect LEDs to P2.0–P2.7 and switches from P1.0/P1.1 to GND.
3. Connect +5 V, GND, oscillator and reset circuitry.
4. Load the Keil-generated HEX file into the AT89C51 component.
5. Set the Proteus clock frequency to 11.0592 MHz.
6. Run the simulation and test all four switch combinations.
7. Capture the complete circuit and one screenshot for each operating mode.

## Verification Record

| Test | Expected output | Verification |
|---|---|---|
| Forward | LED1 to LED8 | Pending Proteus test |
| Reverse | LED8 to LED1 | Pending Proteus test |
| Bounce | LED1 to LED8 and back | Pending Proteus test |
| Reserved | All LEDs OFF | Pending Proteus test |

## Applications

- Running-light and decorative LED displays
- Sequential status indicators
- Signalling panels
- Demonstrations of 8051 GPIO, bit rotation and lookup-table access

## Result

The LED Motion Engine has been documented and an 8051 Assembly implementation has been added. The expected forward, reverse, bounce and reserved-mode outputs are specified above. Successful hardware or Proteus operation must be confirmed by assembling the source, loading the generated HEX file and recording the simulation output.

## Project Status

- [x] Experiment documentation
- [x] 8051 Assembly source added
- [ ] Keil assembly build verified
- [ ] HEX file generated
- [ ] Proteus circuit simulated
- [ ] Output screenshots captured
