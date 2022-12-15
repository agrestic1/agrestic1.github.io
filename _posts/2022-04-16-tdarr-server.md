---
layout: post
title: "Tdarr server and node configuration to convert Videos to h265"
date: 2022-04-16 10:00:00 +200
categories: homelab 
tags: [tdarr,docker,hevc,h265,nvidia]
---

From [https://docs.technotim.live/posts/tadarr-server/](https://docs.technotim.live/posts/tadarr-server/)

[![I Freed Up 700GB+ Converting my Videos Using Tdarr](https://img.youtube.com/vi/UA1Sktq40pA/0.jpg)](https://www.youtube.com/watch?v=UA1Sktq40pA "I Freed Up 700GB+ Converting my Videos Using Tdarr")

Tdarr is a distributed transcoding system that runs on on Windows, Mac, Linux, Arm, Docker, and even Unraid.  It uses a server with one or more nodes to transcode videos into any format you like.  Today, we'll set up the Docker and Windows version of Tdarr using a GPU to regain up to 50% of your disk space back.  I converted my video collection using Tdarr to h265.  

📺 [Watch Video](https://www.youtube.com/watch?v=UA1Sktq40pA)

## Howto
HowTo:

Ordner temp und media in den docker container mounten. In tdarr eine Library erstellen, auf lokalen ordner verweisen "/tdarr_convert", Transcode cache "/temp" und GPU aktivieren.

## Docker Server + Node

`docker-compose.yml`

```yml
version: "3.4"
services:
  tdarr:
    container_name: tdarr
    image: ghcr.io/haveagitgat/tdarr:latest
    restart: unless-stopped
    network_mode: bridge
    ports:
      - 8265:8265 # webUI port
      - 8266:8266 # server port
      - 8267:8267 # Internal node port
    environment:
      # - TZ=America/Chicago
      - PUID=1000
      - PGID=1000
      - UMASK_SET=002
      - serverIP=0.0.0.0
      - serverPort=8266
      - webUIPort=8265
      - internalNode=true
      - nodeID=MyInternalNode
      - nodeIP=0.0.0.0
      - nodePort=8267
      - NVIDIA_DRIVER_CAPABILITIES=all
      - NVIDIA_VISIBLE_DEVICES=all
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /volume1/docker/tdarr/data/server:/app/server
      - /volume1/docker/tdarr/data/configs:/app/configs
      - /volume1/docker/tdarr/data/logs:/app/logs
      # - /volumeUSB1/usbshare/Filme/:/media
      # - /volumeUSB1/usbshare/Bilder/:/bilder
      - /volumeUSB1/usbshare/tdarr_convert/:/tdarr_convert
      - /volumeUSB1/usbshare/temp:/temp
      # - /volume1/homes/Martin/tdarr_convert/:/tdarr_convert # funktioniert nicht, Medien auf volume1 werden in tdarr im Browser der library nicht angezeigt, obwohl im container vorhanden (docker exec -it)
      # - /volume1/homes/Martin/temp:/temp
    devices:
      - /dev/dri # For Intel grafic card and QSV
      
    # deploy:
    #  resources:
    #    reservations:
    #      devices:
    #        - capabilities:
    #          - gpu
```

## Windows Node

Am Rechner müssen Laufwerke mit diesem Ordner ebenfalls gemountet sein und in Tdarr_Node_Config.json für den Computer richtig translatet, siehe unten.

`Tdarr_Node_Config.json`

```json
{
  "nodeID": "MSI",
  "serverIP": "100.125.34.70",
  "serverPort": "8266",
  "handbrakePath": "",
  "ffmpegPath": "",
  "mkvpropeditPath": "",
  "pathTranslators": [
    {
      "server": "/tdarr_convert",
      "node": "Y:/tdarr_convert/"
    },
    {
      "server": "/temp",
      "node": "Y:/temp"
    }
  ],
  "platform_arch": "win32_x64_docker_false",
  "logLevel": "INFO",
  "nodeIP": "100.111.213.103",
  "nodePort": "8267"
}
```
