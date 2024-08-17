---
title: 'Home Server : My self-hosted services'
description: Discover how I host multiples services on my own server
date: {}
categories: []
media_subpath: /assets/img/posts/homeserver
tags: []
lang: en
---

In a world, where we are more and more dependent to the cloud and large corporations to store our data, I became quickly interested in self-hosting. For 3 years, I host several services on my server for my relatives and me.

# The start of my journey: Raspberry PI and PI-Hole 

My self-hosted addiction first started with a Raspberry PI and the PI-Hole application to filter all advertisement and tracking in my network at a DNS level. I used it like that for almost a year, but then I wanted to have more. 

IMAGE PIHOLE RPI

The first service I wanted to add was a media server. The most popular at that time was Plex. 
So I connected a 1 TO hard drive to store the medias and tried to install and configure Plex Server. It was relatively easy to set up but after using it for some times, I encountered some issues. 

First, instabilities occured regularly when Plex or other software was updated. 
Second, the Raspberry PI model 3B+ with 1GB RAM and a quad-core processor cadenced at 1.4GHz, was struggling to decode high-resolution media. 

To fix this issues I took two initiatives : using Docker and buying new hardware.

# Easy setup with Docker

Docker is a platform designed to run software inside a container. Containers are isolated from one another and bundle their own software, libraries and configuration file. And because all the containers share a single operating system kernel, they use fewer resources than virtual machines. 

![Docker](docker.png){: w="400" h="150"}
_Docker_

For those reasons, using Docker is an ideal choice and it becomes trivial to manage multiples services running on the same server and not think about dependencies, incompatibilities, etc...

Also, I define all my container configuration in YAML files by using Docker Compose. This way all the server configurations can be easily backed up and updated.

# A hardware more powerful

With more and more services and my RPI inability to decode some large media, I choose to buy a more powerful hardware. I found a Lenovo Thinkstation on the second hand market (which is often flooded with old enterprise hardware). It is a lot cheaper than a real server and is largely enough for my use.

IMAGE SERVER

For the OS, I installed the latest stable Debian release (headless version) to have a solid fundation and still have compatibility with most software.

It was installed on a new 512Go SSD to improve the performance. The 1 TO hard drive store only services data (media, static storage, ...).

# My server services

Now that I had a nice and running server, I started to add more and more services depending on my needs. I made the following infographic with the services I am currently using :

![Homeserver Architecture](beniserv.png){: w="400" h="150"}
_My Home Server Services_

## Access and security

The most important service is probably Caddy. It is a web server with automatic HTTPS and reverse proxy feature. 

In other word, thanks to Caddy, I can access my services directly by using subdomain and all the trafic will be encrypted and forwarded to the 443 port. For example, I could access one of my service directly on "myservice.beniserv.fr" and another on "myotherservice.beniserv.fr", etc...

With this setup, the only port I have to expose one port (443) to internet, and everything is always encrypted.

I also added a dynamic DNS plugin to Caddy, in order to sync my IP server adress with my domain name provider. So even if my internet service provider attributes me a dynamic IP, I can always have access to my homeserver via my domain name.

To improve even more the security, I also use a service called Crowdsec to detect peers with malicious behaviors and block them for accessing my server. Crowdsec analyses the Caddy logs and if its behavior engine detects one of the configured scenario, it will blocks the IP at a firewall level. The avantage of Crowdsec over the other similar services available (Fail2ban for example), is that it offers a collobarative solution. Indeed, when a Crowdsec user blocks an aggressive IP, it is also shared among all users to further improve everyone's security.

## Update and backup

I also have Watchtower to always keep all my services updated to always benifits from the latest security fix and features. Every night, it downloads the latest versions and replace them if an update is available.

And because sometimes an update or a storage drive can fail, I use a service to do a periodic backup. Every night, it will save all the server configurations and some data in a encrypted archive file. This archive is then saved on my 1 TO harddrive and to a AWS cloud storage server. A retention policy of 7 days make sure to not bloat the backup storage by removing the oldest backup archives.

## Monitoring

With multiples services running, I need to have a way to monitor them and receive notification when an issue is detected. That's why I use Portainer and Homepage.

Portainer allows to directly manage docker containers via a graphical interface. Even though I prefer to always use Docker Compose to deploy new service, I still use Portainer to monitor the health status of my already existing services. When needed, I can also easely restart them from the Portainer UI.

Homepage, is an highly customizable dashboard. It proposes widgets for a large choice of services, and can display differents system metrics (uptime, disk space, server temperature, ...). It is a fancy way to access my differents services.

TODO IMAGE HOMEPAGE

To be alerted in case of errors, my services are sending messages on a private Discord server.

## Other services

Like you can see on the previous image,  the following services are also running :
- **Jellyfin** : Media server I use instead of Plex.
- **Paperless-ngx** : Document management and storage system
- **Nextcloud** : Cloud storage service
- **QBitorrent** : Torrent client
- And more...
