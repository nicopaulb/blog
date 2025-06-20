---
title: "Garmin DeLijn Tracker : Bus tracker widget for Garmin watches"
description: "A simple bus tracker widget compatible with Garmin watches to follow all DeLijn buses in Flanders."
date: 2025-01-30
categories: [Embedded]
tags: [c, network, garmin, watch]
media_subpath: /assets/img/posts/garminDeLijn
lang: en
image: 
    path: garmin_screen.png
---

A year after my last Bus Tracker project for bus in Toulouse, I moved to Belgium and decided to create a similar tool but for Garmin watches. 
I personnaly wear a Forunner 245 and always wanted to be able to know exactly when the bus is coming to pick me up.
So I created this app to track all DeLijn buses in Flanders and display the time of arrival of the next bus.

# DeLijn Bus Data API

The first step was to get access to the DeLijn API to receive all the realtime bus information.
Fortunately, it is free to use and only require to create an account on the [De Lijn Open Data portal](https://data.delijn.be/).

Once my keys in hand, DeLijn provided three differents APIs :
- GTFS Static :  Standart API to get static public transport info
- GTFS Realtime : Standart API to get realtime public transport info
- Open Data Services : Custom API to get realtime and static public transport info

So from this brief description, you can deduce that to get realtime information, it is possible to use either the GTFS Realtime or the Open Data Services API. 
But before telling you which one I choose to use, let's take a look at the difference between the two APIs.

## GTFS

General Transit Feed Specification (GTFS) is a standard format for public transport schedules created by Google in 2005. Initially created to incorporate transit data into Google Maps, it has since been adopted by most other navigation services (Apple Maps, Moovit, ...) and public transit services.
It allows transit agencies to publish their schedule, route, and stop data in a format that navigation services can easaly integrate.

A GTFS feed is a collection of files that describe the public transport network, including routes, stops, and schedules. However a GTFS feed is not enough to get realtime information because it contains only the static/planned schedules and not the realtime delays or trip updates.

To have this additional realtime information, you need to fetch another feed, GTFS-RT (GTFS Realtime), which is an extension of GTFS. This feed will contains only relative delays for each trip and not the absolute arrival time. So you need to combine the results of both GTFS APIs to compute the absolute arrival/departure time of public transport vehicles.

> See [GTFS official website](https://gtfs.org) for more information.
{: .prompt-tip }

## Open Data Services

The Open Data Services API is a custom API by DeLijn that provides realtime and static public transport information. It is not standart but offers the same information as the GTFS APIs in a JSON format and does not require to make additional requests to fetch the realtime information.

> See [DeLijn Data website](https://data.delijn.be/product#product=5978abf6e8b4390cc83196ad) for more information.
{: .prompt-tip }

On a embedded environnement with limited memory and processing power, the Open Data Services API seemed to me the best choice. The response being a JSON object, it can be easely parsed and does not require to navigate between differents files and execute differents HTTP requests like the GTFS APIs.

# Garmin SDK

To develop an application for a Garmin product, you have to use the Garmin [Connect IQ SDK](https://developer.garmin.com/connect-iq/overview/).
The SDK allows to build native applications, widgets and data fields for all Garmin smartwatches. 

## C-Monkey

Garmin developed its own programming language called *Monkey C* to use the SDK. The syntax is derived from C, Java, Javascript, and is quite easy to understand.

IMAGE MONKEY C

> See [Garmin Monkey C](https://developer.garmin.com/connect-iq/monkey-c/) for more information.
{: .prompt-tip }

One of the first thing I made is to install the official Monkey C language support extension for VS Code. As well as offering syntax highlighting and code completion, it also ease the creation of a new project by providing a few useful commands (build, open samples, open SDK manager, ...).

## Emulator

The SDK manager also comes with a device emulators to directly run the application from the computer .
It is quite handy to quickly run the application without having to install it on the watch or to be able to test it on differents watches.

IMAGE EMULATOR


# Application

## Architecture
The application is quite simple, it fetchs the time of arrival of the next bus from the DeLijn Open Data Services API and displays it a countdown on the watch.
The countdown is updated every second and a new API request is made every minute or, if the refresh button is pressed, to correct the countdown if the bus is delayed or advanced. This way, the informations displayed are always up to date.

IMAGE SCHEMA

## Settings
In the settings, accessible from the Connect IQ application, you can select which bus stop and bus lines you want to track. You should also set your own DeLijn API key, so you don't have to worry about rate limits.

IMAGE SETTINGS

The API request interval can be changed as well. 

## Interface
The countdown (in minutes) to the next bus is displayed.
The interface can shows up to two bus stop information, one on each line.

IMAGE INTERFACE

> The code is available on Github here : [https://github.com/nicopaulb/Garmin-DeLijn-Bus-Tracker](https://github.com/nicopaulb/Garmin-DeLijn-Bus-Tracker).
{: .prompt-tip }

# Garmin Connect IQ Store

Publishing the application was quite straightforward if you compare it to a lot of others stores and platforms. Simply create an account, upload the application, update the store details and you're done.

> The Garmin application is available on the Connect IQ Store here : [https://apps.garmin.com/fr-FR/apps/1d2b5826-ae2e-4bb9-a6e7-76e3e6b1ef5a
](https://apps.garmin.com/fr-FR/apps/1d2b5826-ae2e-4bb9-a6e7-76e3e6b1ef5a).
{: .prompt-tip }

