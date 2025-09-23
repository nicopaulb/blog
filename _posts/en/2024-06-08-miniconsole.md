---
title: "Retro handheld : Minimalist game console"
description: "A STM32 board equiped with a mini OLED screen and a Xbox Controller to recreate some old memories..."
date: 2024-06-08
categories: [Embedded]
tags: [ble, reverse, st, c]
media_subpath: /assets/img/posts/miniconsole
lang: en
mermaid: true
---

I was looking for a project idea to test my new **NUCLEO-WB55** evaluation board when my eyes landed on a **Xbox controller** lying on my desk. This controller can connect remotely to a device via a proprietary 5 GHz protocol requiring a USB adapter or by the famous protocol : **BLE (Bluetooth Low Energy)**. 
I also remembered having a small unused OLED display screen in my stock, originally bought for my **Bus Tracker** project. I thought it could be a nice occasion to also test this screen.

> If you want to learn more about my **Bus Tracker** project, you can read more about it [here]({% post_url en/2023-02-10-bustracker %}).
{: .prompt-tip }

Those mini-screen reminded me of the cheap mini handheld game console which was popular when I was a child.

![Mini handled game console](minihandled_model.jpg){: w="200" h="300"}
_Mini handled game console_

Now the goal of my next project was clear : create a similar handheld game console based on a **STM32 microcontroller** with a colorful screen and playable with a wireless Xbox Controller !

## Communication with a Xbox Controller

The first goal of this project is to connect the Xbox controller to my **NUCLEO-WB55 board**. The two communicate via BLE, but an appropriate driver should be written for the microcontroller to realize the connection and parse the controller inputs. 

So before starting programming on my board, I need to know how the controller communicate with the Xbox driver for my PC. 

To do that I am helped by two software :
- **Wireshark** on Windows/Linux, a network packet analysis software which can also capture and dissect BLE packets
- **nRF Connect** on Android, a generic BLE tool to scan, advertise, and communicate with a device.

### Analyzing BLE packets via Wireshark

By connecting the controller to my PC and intercepting the BLE packets between the two devices, I can see all the frame sent by the controller when I realize a specific action (joystick moved, button pressed, trigger pushed, ...).

For example, if I press the button 'B', the controller send the following frame : 

![BLE packet received when button B is pressed](xbox_ble_packet.png){: w="1000" h="700"}
_BLE packet received when button B is pressed_

The most important part of this packet is the **BLE attribute value**, containing information about all the **controller inputs**.
So, I filled an **Excel document** with the attribute value intercepted for each performed action and split it by byte. Note that to make it easier to **reverse the protocol**, I only do one action at a time. 
This way I am able to identify the purpose of each byte field in the BLE attribute value. 
Once all the packets captured, I deduced the following table :

![Table of packets for each action](xbox_parse_table.png){: w="1000" h="700"}
_Table of packets for each action_

### Writing a Wireshark plugin

To check I didn't do any mistake and to better visualize the packet received, I developed a **Wireshark dissector**. Dissectors are meant to analyze some part of a packet's data and I choose to integrate one into Wireshark via a **plugin**.

I did as followed : 
- I **downloaded the Wireshark source code** and installed all the compilation tools
- I **wrote my plugin code in C** by following the [Wireshark documentation on dissectors](https://github.com/wireshark/wireshark/blob/master/doc/README.dissector)
- I **recompiled the application** and the plugin together

The **Xbox controller protocol** is then automatically detected when a BLE packet coming from the controller is received.

![Wireshark with the Xbox Controller Dissector](xbox_wireshark_packet.png){: w="1000" h="700"}
_Wireshark with the Xbox Controller Dissector_

> My Xbox controller dissector for Wireshark is available on Github : [https://github.com/nicopaulb/xbox-wireshark-dissector/tree/main](https://github.com/nicopaulb/xbox-wireshark-dissector/tree/main)
{: .prompt-tip }

### Analyzing BLE profile via nRF Connect

On my phone I launched **nRF Connect** to discover the BLE services and characteristics defined by the controller. I noticed it had a service called **HID Report** and guessed it was somehow using the **HID protocol** (designed for USB device) over BLE.


// IMAGE nRF CONNECT

After a quick search on internet, I found there is an official **BLE profile** called [**HID over GATT**](https://www.bluetooth.com/specifications/specs/hid-over-gatt-profile-1-0/)

#### HID Protocol

The **Human Interface Device (HID) protocol** is a standard used by USB devices like keyboards, mice, game controllers, and touchscreens to communicate with an host system efficiently. This way the host operating system can include a built-in and standardized HID driver able to interpret any input devices.

During the device enumeration phase, a **HID descriptor** containing informations about the type of device and features (number of buttons, axes, keystrokes, ...) is sent to the host. Also a **report descriptor** describing the format of data packets (**reports**) and how they should be interpreted can be asked by the host.

The HID host determines how often the device should send data by periodically polling the device at a fixed interval for **input reports** (key presses, mouse movement, ...). The device can also receive **output reports** from the host to for example setting a LED indicators on a keyboard or enabling controller rumble. 

#### HOG (HID over GATT)

The **HID over GATT profile** is a way to use HID protocol over BLE. It is based on GATT (Generic Attribute Profile) and define an HID service with characteristic which are in turn based on the HID descriptors.

## STM32WB55 programming

### STM32CubeIDE

To start programming the STM32WB55, I used STM32CubeIDE.

### Architecture

#### BLE stack

- "Just work" pairing method (io capabilities = no-display/keyboard, mitm off), no security

```mermaid
sequenceDiagram
  participant A as Application<br>(STM32WB55)
  participant B as BLE Stack<br>(STM32WB55)
  participant X as Xbox Controller

  A <<->> B: Initialize GATT & GAP
  alt Search a Xbox Controller
    A ->>+ B: aci_gap_start_general_discovery_proc
    B ->> X: Start scan
    X ->> B: Advertising<br>(Complete Local Name = Xbox Wireless Controller)
    B ->> A: HCI_LE_ADVERTISING_REPORT_EVENT
    A ->> B: aci_gap_terminate_gap_proc
    B ->>- A: ACI_GAP_PROC_COMPLETE_EVENT
  end
  alt Connect to the found controller
    A ->> B: aci_gap_create_connection
    B ->> X: Connection request
    X ->> B: Link etablished
    B ->> A: HCI_LE_CONNECTION_COMPLETE_EVENT
  end
  alt Bound to the controller<br>('Just Work' method)
    A ->> B: aci_gap_send_pairing_req
    B ->> X: Pairing request
    X ->> B: Pairing response
    B ->> A: ACI_GAP_PAIRING_COMPLETE_EVENT
  end
    A <<->> B: GATT procedure
```
<p style="color:#6d6c6c;font-size: 80%;text-align:center;">Connection and pairing sequence diagram</p>

#### TFT display API

#### Menu and navigation

#### Testing screen

#### First game : Snake


## PCB design

- Designed a Nucleo-WB55 hat with the screen and a passive buzzer for sound. Convenient for protoype because can be directly plugged in the development board.
- Used Fusion 360 design tool to create the necessary component schematic and footprints, create the PCB layout and route it.
- Printed the PCB via JLPCB and soldered the components (heading pins, TFT display and passive buzzer).
