---
title: 'beepify.io - implementation plan'
date: Sun, 16 Oct 2016 15:00:21 +0000
draft: false
tags: ['beepify.io', 'plan', 'selfhackaton']
---

This is the first implementation plan :) that will of course change during the implementation. So this is only the first guideline. - release 1 server create the models - read current location latitude, longitude (will need this to test) - add sample data to the database create services - get data receive product code, latitude and longitude return all prices deploy application to server client enter product code get current location from GPS get product prices from server - release 2 client read barcode server restrict the list to 1km range - release 3 client show stores in a map - release 4 server - add data service client - if product not found add data