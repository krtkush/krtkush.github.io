---
title: "My Selfhosted Setup - Part 1"
date: 2024-10-02
author: Kartikey Kushwaha
layout: post
permalink: /my-selfhosted-setup-part-1/
categories: [Setup]
comments: false
tags: [Setup]
---

For the past one year I have been setting up my own self-hosted services and servers. 
It has come to a point were most of it is not in automatic mode. 

Here I try to document my setup; mostly for my own reference in the future.

# Humble beginnings

It started with a need for a pi-hole and a home for my Homebridge server which, at that moment, was running on my M2 Mac Mini. 
As I started setting up the server(s) and discovering new tools and services, I realised that there is a lot more 
potential to do fun stuff and learn.

My apartment is pretty small, so I didn't need any sophisticated hardware to make internet accessible throughout. 
However, the Vodafone Station is a painful hardware to deal with as it barely allows any user configuration and 
was frequently crapping out under the load of 20 or so WiFi devices. I had to turn it into "Bridge mode" and plug in a 
TP Link router for everything to work as expected.

Next, I bought a Raspberry Pi 5, attached a NVMe SSD, flashed the default RPi OS on it and hooked it up to the router via ethernet.

## Server 1: Home Server (Germany)

Configuration - 

1. Raspberry Pi 5 with 8 GB RAM
2. 500 GB + 128 GB SSD storage via a duel NVMe HAT
3. Official Active Cooler

Things I am running on the Home Server - 

1. Pi-hole
2. Home bridge
3. Home Assist
4. File Browser
5. Syncthing
6. Cockpit
7. Portainer
8. Dash
9. Homarr
10. Nginx Proxy Manager
11. Tailscale 
12. Watchdog
13. Keepalived
14. Docker

Majority of the apps above run in their own Docker Compose container which are easily managed via Portainer.

# Tailscale

Discovering Tailscale meant that a whole new world of possibilities opened up in front of me. 
I realised I could have my own VPN by setting up a device at my parent's home in India and 
configure it as a Tailscale exit node. This way, I could have access to geo-locked content streaming services 
(Disney+ Hotstar, Jio Cinema etc.), cheap Netflix subscription prices and password sharing with my parents. 

Hence, I bought another RPi 5, set it up with pretty much the same stuff as the home server, minus the 
home automation services and pi-hole but plus, qBittorrent ;) At my own home, I installed Tailscale on my Apple TV and 
enable it to use Remote Server as it's exit node whenever I need to access Indian streaming content.

And finally, the Home and Remote server share each other's Syncthing sync directory. 
This way I can easily transfer files between the servers reliably. In fact, the Remote Server's qBittorrent directly
downloads files into the sync folder which then automatically syncs with my Home Server. I never torrent in Germany 
because of the horror stories I have heard.   

## Server 2: Remote Server (India)

Configuration -

1. Raspberry Pi 5 with 4 GB RAM
2. 500 GB SSD storage via a NVMe HAT
3. Official Active Cooler

Thankfully, my dad is adept at managing tech hardware. He makes sure things are running smoothly 
and keeps track of internet connection, ambient temperature etc.

# Pi-hole backup

For some reason my home server stops responding after 2-5 days of being online and I have to restart it manually 
or rely on watchdog to catch the issue and restart it. I suspect that it is the duel NVMe HAT or the drive itself
but the exact cause is something I have yet to figure out. Pi-hole being up 24*7 is important for me because
without it, the home internet dies. Hence, I bought a RPi Zero and set it up as a backup pi-hole using 
keepalived and following [this](https://davidshomelab.com/pi-hole-failover-with-keepalived/) tutorial.

Now, whenever the Home Server goes down, the RPi Zero takes over instantly without disruptions. 
However, there's a limitation: I use Tailscale and pi-Hole to filter traffic on my mobile devices, even when away from home. 
Since Tailscale has no equivalent backup service, if my RPi goes down while I'm away, my internet access is disrupted. 
It usually takes me a while to realize I need to disable Tailscale. Additionally, The Pi-Zero, is not proving to be a stable device.       

## Server 3: Mini Home Server
1. Raspberry Pi Zero 2 W
2. 64 GB SD Card

Over the months I have realised that for some reason it is not a very reliable device; making the Pi-hole keepalived setup useless. 
Thankfully, the RPi 5 is becoming more stable, so I might not really need the Zero.   









