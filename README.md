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
The default Shelly with the base Home Assistant configuration can perform these functions, but it requires workarounds and helpers that make it a less than ideal solution.  Instead, ESPHome provides additional safeguards and deterministic settings that ensure the device reads and issues commands according to specifics of the High Spec Connector.  ESPHome can present these sensors correctly to Home Assistant.

# Installation
