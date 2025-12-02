---
title: "Room Mapper : Take measurements with a LIDAR"
description: ""
date: 2025-11-20
categories: [Embedded]
tags: [st, c]
media_subpath: /assets/img/posts/roommapper
lang: en
image:
  path: cover.jpg
mermaid: true
---

- Choix du LIDAR (lien vers page github avec la liste)

## Hardware

# STM32

- STM32 black pill (STM32F411CEUX)
- Programming/debugging using SWD with ST-LINK from Nucleo board.

# RPLIDAR

# Display

- Tactile TFT display :
	- ILI9488 chip to control the display
	- XPT2046 chip to handle touch

# Sound


## Software

### Architecture

### Implementation

#### RPLIDAR Driver

- Driver development utilisant le STM32 HAL :
	- UART
	- Protocol décrit dans doc (mettre lien et résumer)
	- Pas de sync bytes, important de ne pas rater un seul byte pour rester syncroniser
	- DMA en RX pour traiter toutes les mesures suffisaement rapidement
	- Functions blocking or non-blocking selon les besoins
- Lien vers github repo du driver
- Unit test for parsing

#### Touch Display Driver

- Driver developement :
	- Inspiré de library dispo ici (mettre lien)
	- Adapté pour memory usage, removed TouchGFX support (ST display library)
	- 2 SPI
	- Interrupt pour touch event
	- Calibration du XPT2046 nécéssaire

#### PWM Buzzer

- Used the buzzer driver code from the miniconsole project to play sound on menu interaction.
- I had to double the duration because of the maximum prescaler value and clock speed.
- First try to use linear then logarithmic scaling for volume but finally jsut hard coded some values in array

#### Menu

#### Mapping LIDAR measurements

- Scale options (auto or manual)
- Quality threshold option
- Show quality linear gradient values
- Persistency option
- Measurement between selected points

#### RPLIDAR diagnostics

- Show heath, device info and samplerates

## Integration

### Schematic

### PCB

## Demo
