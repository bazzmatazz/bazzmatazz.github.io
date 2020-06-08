---
layout: post
title:  "Jetson Nano Flash"
date:   2020-06-07 01:00:23 +0500
categories: 
---
- Would need:
  - A microSD card. At least more than 16GB. 
  - Device to flash a microSD card && a microSD card reader
  - Jetson nano

- [Download Nano Dev Kit SD Card Image](https://developer.nvidia.com/jetson-nano-sd-card-image)
- Unzip the image `unzip nv-*.zip`

- Install etcher
  - https://www.balena.io/etcher/
  - unzip balena-etcher-electron*.zip
  - ./balenaEtcher-1.5.95-x64.AppImage

- Insert SD card
  - *Make sure write lock on the card is on, otherwise the card would be in a read-only mode*

- Select the image and the reader
- Flash
![Flashing](../images/etcher_flashing.png)
