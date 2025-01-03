# Ford Transit Door Interface

This readme will walk you through connecting a Shelly Plus RGBW PM to the Ford Transit's 43-in High Spec Connector to read door open/close state and issue Lock/Unlock commands.  Thie enables the user to integrate door states and lock/unlock commands into Home Assistant for further automation and visability.  ESPHome is loaded onto the Shelly with the appropiate settings to read door states and issue lock/unlock commands.

It requires the following hardware:
- Ford Transit optioned with the High Spec Connector
- 43-pin High Spec Connector Wiring Harness
- Shelly Plus RGBW PM
- USB-to-UART Serial Adapter

### Disclaimer
This guide has been tested on the North America 2021 Ford Transit, but is likely applicable other model years and geographies.  Some wiring colors will vary based on left-hand vs right-hand driven Transits.  Your mileage may vary.

Lastly, please be advised that the Ford BEMM is not 100% accurate and should only be considered a guide, not the truth.  The following required testing to in order to get working.  

# Introduction

The ability to monitor door open/closed state in Home Assistant can allow the user to trigger automations such as turning on lights or sending notifications upon event.  Additionally, events can be triggered should the door state open or closed for a period of time.  More interestingly, the ability to Lock and Unlock the doors allows further automation durning bed time routines or ability to lock/unlock the van with a phone or smart watch over Wi-Fi when outside of cellular range.   Using just a Shelly RGBW PM is an easy, reliable, and inexpensive method to achieve these capabilities.  With a hour or two of time, one can set this up using the guide below.

### Hardware
The Shelly Plus RGBW PM is an ESP32-based device with 4 inputs and 4 outputs.  The inputs can be triggered with a ground and the outputs are MOSFET ground-switched with 4 amps of output per channel and 10 amps concurrently across the four outputs.  More details can be found here: https://kb.shelly.cloud/knowledge-base/shelly-plus-rgbw-pm

Given this hardware, we can read the door open/closed states on all the van doors (driver/passenger/sliding/cargo) independently using the 4 inputs.  Additiionally, we can trigger lock and unlock commands by ground switching the lock and unlock signal lines.

### Software
The default Shelly with the base Home Assistant configuration can perform these functions, but it requires workarounds and helpers that make it a less than ideal solution.  ESPHome provides additional safeguards and deterministic settings that ensure the device reads and issues commands according to specifics of the High Spec Connector.  ESPHome can present these sensors correctly to Home Assistant. 

This guide will take you through flashing the Shelly and installing it into the van.  Details on how ESPHome runs on the Shelly Plus RGBWPM are found here: https://devices.esphome.io/devices/Shelly-Plus-RGBW-PM

# Installation
Let's start at your desk.  Grab the following:

- Shelly RGBW PM
- 43-pin wiring harness
- USB-to-UART Serial adapter.  If you don't have a serial adapter, I use this one: https://www.amazon.com/dp/B07WX2DSVB
- A small flathead screw drawer (for the wire terminals)
- Your laptop
- A cup of coffee or tea.

### Connect the Shelly to UART
The Shelly Plus RGBW PM has a small 7-pin connector on the bottom.  This interface exposes the UART interface of the ESP32.  We need to connect to this interface to flash the Shelly.  Unfortnately the pins on this connector are extremely small, 1.27mm to be exact.  Your run of the mill 2.54m jumper pin won't fit.  Instead you'll need to subtitute the pin with a thin solid wire to act a jumper.  Fortunately there are two household items that can work:

- A cat5e cable with solid wire cores.
- A twisty tie

I opted for the second option.  Strip the twisty tie and cut into 1" lengths for a total of 5 pieces. Those into the following:

- GND (Ground)
- GPIO0
- +3.3_ESP (Positive)
- U0RXD (RX)
- U0TXD (TX)

![image](https://github.com/user-attachments/assets/c2aa4c67-f9d2-4efa-b1b7-a312b57776d6)

Use a jumper wire and tie GPIO0 to Ground.  I simply ran a small cable to the ground terminal on the Shelly.  This is needed at the top of boot to place the ESP32 into the bootloader.

Connect the remaining wires to your UART adapter.  Remember to transpose the TX/RX wires:

- UART TX >> Shelly RX
- UART RX >> Shelly TX

