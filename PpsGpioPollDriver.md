The ar71xx/ath79 devices does not seem to have interrupts on GPIO pins. The `pps-gpio-poll` driver polls the GPIO input with the following procedure:

  * Poll the GPIO every 100 us.
  * When the input changes from low to high, schedule a busy wait (1s - 150us) in the future

The busy wait reads the GPIO input in a loop, waiting for a low to high transition. When the transition happens:

  * Capture the PPS timestamp
  * If this is the first captured timestamp, schedule the next busy wait (1s - 50us)
  * Otherwise, schedule the next busy wait 15us ahead of the next predicted transition.
If the busy wait is scheduled late and the transition event is not captured in the loop, the driver returns to polling mode.

## Module Parameters ##
| **Parameter** | **Description** | **Default** |
|:--------------|:----------------|:------------|
| gpio | GPIO number | 0 |
| capture | capture the rising edge if 1, falling edge if 0 | 1 |
| poll | poll interval (us) | 100 |
| wait | start waiting ahead of the next GPIO transition (us) | 15 |
| debug | Show debug messages | 0 |