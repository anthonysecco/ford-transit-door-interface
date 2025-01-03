# Ford Transit Door Interface

This readme will walk you through connecting a Shelly Plus RGBW PM to the Ford Transit's 43-in High Spec Connector to read door open/close state and issue Lock/Unlock commands.  Thie enables the user to integrate door states and lock/unlock commands into Home Assistant for further automation and visability.  ESPHome is loaded onto the Shelly with the appropiate settings to read door states and issue lock/unlock commands.

It requires the following hardware:
- Ford Transit optioned with the High Spec Connector
- 43-pin High Spec Connector Wiring Harness
- Shelly Plus RGBW PM
- USB-to-UART Serial Adapter

### Disclaimer
This guide has been tested on the North America 2021 Ford Transit, but is likely applicable other model years and geographies.  Some wiring colors will vary based on left-hand vs right-hand driven Transits.  Your mileage may vary.

The end of this guide includes an extra section on start/stop inhibit.  This requires moving a wire to empty pin on the 43-pin connector and is only applicable to Transit with Stop/Stop engine capabilities.

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

### Connect the Shelly to UART Adapter
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

Connect the remaining four wires to your UART adapter.  Remember to transpose the TX/RX wires:

- UART Ground >> Shelly Ground
- UART Positive >> Shelly Positive
- UART TX >> Shelly RX
- UART RX >> Shelly TX

No other wires are needed.  The UART adapter will power the ESP32 on its own.

### Flash the Shelly with ESPHome

With everything wired, connect the UART adapater to your PC.  This will power the Shelly.  If you have grounded your GPIO0 pin, the LED on the Shelly will be solid on (not flashing).  This indicates the bootloader is running and ready for flashing.

On Home Assistant, create a new device in the ESPHome compiler.  Use the YAML provided here:

When saving, select "Plug into this computer" and follow the instructions.  Download the bin file, then head to "Open ESPHome Web" to flash.  Click connect, select the COM port associated to your UART adapter.  If you did everything right, you'll end up with a screen below:

![image](https://github.com/user-attachments/assets/3d805851-3c22-4ff4-92c3-ff66259eda01)

Select "Install" and select the bin you just downloaded.  It will begin flashing and all goes well it'll take 2 minutes to complete.  If you run into an issue, make sure your RX/TX pins are correct.

Once complete, remove the ground wire on GPIO0.  Then, remove and reconnect the USB to UART adapter.  This will power cycle the Shelly.  Upon start, the LED should be flashing indicating the AP is up.  Using your computer or phone, check to see if the network "transit-door-interface" is being broadcasted.  

If so, your flashing was successful.  Disconnect the UART, remove the wires and pins from the Shelly and proceed to the next step.

### Connect the 43-pin wiring hardness to the Shelly Plus RGBW PM

With a pair of wire strippers, clip and strip the following wires and connect them to the Shelly wiring terminals.  The stripped length should only be enough to minimize exposed wire outside the Shelly.  Connect the wires according to the table below

| Description | Shelly | Wire |
|---|---|---|
| Driver Door State | I1 | Green w/ Violet Stripe |
| Passenger Door State | I2 | White |
| Sliding Door State | I3 | Yellow |
| Cargo Door State | I4 | Brown w/ Violet Stripe |
| Lock Command | O1 | Gray w/ Yellow Stripe |
| Unlock Command | O2 | Violet w/ Gray Stripe |
| Start/Stop Inhibit | O3 | |
| Unused | O4 | |

With that completed we can head to the van.

### Connecting the High Spec Connector

Open the glovebox and empty it of contents.  Push in the sides so the glovebox falls forward exposing the innards of the what's behind the dash.  On the right side you'll find the High Spec connector and the butt connector.  Remove the latch and the butt connector.  Insert the wiring harness connector and pull the latch closed.  You're now connected to the van.  Now for power.

### Powering the Shelly

It is possible to power the shelly using pin #28 on the 43-pin High Spec Connector for +12v and ground the C33-H connector.  That said, the Shelly (err ESP32) draws ~100ma constantly.  This small, but consistent, draw will slowly drain the van battery.  It also requires Fuse F15 to be placed in always on mode.  As such, it's preferable to connect the Shelly's Postive and Ground terminal to the house battery system.  

I simply ran an 18/2 wire under the floor near the center and then up and Behind the glovebox.  This was straight forward and completely hidden.  With the Shelly connected this way there's no worry to drain the van battery.  Switch the power source on and your Shelly should boot up with a flashing LED.

### Final configuration and testing
Using a laptop or phone connect to the Shelly access point.  It should be called "transit-door-interface" using password "12345678".  Once connected, it should lead you to a captive portal that you will use to connect it to the Wi-Fi network in the van.  Once connected, go to Home Assistant and ESPHome to adopt the device.

Once adopted you should see the following sensors under the device:

![image](https://github.com/user-attachments/assets/aefb2c9b-f73b-487b-82a6-0f716b084877)

Open and Close the four doors to ensure the state change is captured.  The reading is instant.  Press lock / unlock buttons to ensure the locks trigger accordingly.

You're done!

# Conclusion

There are a number of interesting automations that can include these sensors.  Some examples are below.

- Turn on specific lights based on which door opens (driver, passenger, sliding, or cargo).
- Turn off heat or AC if van doors open for greater than X minutes.
- Send notification to mobile when door is opened you're away from the van.
- Send Lock/Unlock commands from phone or smartwatch when you don't have keys on you.  Can be done with no cellular if you're nearby and on the van Wi-Fi.
- Send Lock command as part of bedtime script

### Limitations

- The lock / unlock functions behavior like the buttons on the doors in the van.  It does not flash the lights on the outside nor enable the alarm.  If you want that behavior, use the FordPass app to lock/unlock.
- It's not possible to determine the lock or unlock state.  It's only possible to trigger the commands to lock or unlock.  This is why the sensor is exposed as a button and not a lock.
- The cargo doors are treated as one entity for open/closed.

### Start/Stop Inhibit

Start/Stop Inhibit can be useful to keep the engine running whilest the house battery is charging from DC-DC converters.  A simple script such as if battery >95%, then set to inhibit switch to true can be useful.  Also, some drivers may prefer to permanetly defeat start/stop out of personal preference.  Use the following to enable this feature.

To enable Start/Stop inhibit, you'll need to:
- Move an unused wire in the 43-pin connector and re-pin it to position #24.
- Connect that wire to O3 on the Shelly.
- Switch the Inhibit switch to 'on' in Home Assistant to Inhibit Start/Stop

### Engine Monitoring

It's possible to monitor the Transit Engine On/Off state by monitoring ground on the Brown w/ Yellow Stripe wire (pin #29).  By adding a Shelly Plus Add-on, I believe this could be connected to the Available GPIO pin on the Shelly.  Additionally, it's possible to add another Shelly Plus RGBW PM or a Shelly i4 DC for this purpose.

### Feedback

If you have feedback or suggestions, don't hesitate to reachout.
