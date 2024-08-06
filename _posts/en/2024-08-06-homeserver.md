---
title: "Home Server : My self-hosted services"
description: "Discover how I host multiples services on my own server"
date: 2024-06-08
categories: []
media_subpath: /assets/img/posts/homeserver
tags: [] 
lang: en
---

In a world, where we are more and more dependent to the cloud and large corporations to store our data, I became quickly interested in self-hosting. Now and for the last 3 years, I host several services on my own server.

# The start of my journey: Raspberry PI and PI-Hole 

My self-hosted addiction first started with a Raspberry PI and the PI-Hole application to filter all advertisement and tracking in my network at a DNS level. I used it like that for almost a year, but then I wanted to have more. 
The first service I wanted to add was a media server. The most popular at that time was Plex. 
So I connected a 1 TO hard drive to store the medias and tried to install and configure Plex Server. It was relatively easy to set up but after using it for some times, I encountered some issues. 

First, instabilities occured regularly when Plex or other software was updated. 
Second, the Raspberry PI model 3B+ with 1GB RAM and a quad-core processor cadenced at 1.4GHz, was struggling to decode high-resolution media. 

To fix this issues I took two initiatives : using Docker and buying new hardware.

# Easy setup with Docker

Docker is a platform designed to run software inside a container. Containers are isolated from one another and bundle their own software, libraries and configuration file. And because all the containers share a single operating system kernel, they use fewer resources than virtual machines.

![Docker](docker.png){: w="400" h="150"}
_Docker_

So by using Docker, it becomes trivial to manage multiples services running on the same server and not think about dependencies, incompatibilities, etc...

Also, I define all my container configuration in YAML files by using Docker Compose. This way all the server configurations can be easily backed up and updated.

# A hardware more powerful

With more and more services and my RPI inability to decode some large media, I choose to buy a more powerful hardware. I found a Lenovo Thinkstation on the second hand market (which is often flooded with old enterprise hardware). It is a lot cheaper than a real server and is largely enough for my use.


For the OS, I installed the latest stable Debian release (headless version) to have a solid fundation and still have compatibility with most software.


# My server services


![Homeserver Architecture](beniserv.png){: w="400" h="150"}
_My Home Server Services_


I said it was an addiction because of how quickly you can add new services once the server is correctly configured, and how satisfying it is to use yours own server everyday.