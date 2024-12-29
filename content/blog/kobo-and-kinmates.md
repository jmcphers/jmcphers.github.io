
---
layout: post
title:  "Kobo and the Kinmates T01"
date:   2024-12-28 21:04:43
categories: hacking
---

As a Christmas gift a few days ago, I received a Kinmates T01 Bluetooth page-turner, branded in the typical logorrheic Amazon fashion as *Tiktok Scrolling Ring, Page Turner for Kindle App, Remote Control for TIK Tok and Kindle App, Bluetooth Connected, for iPhone Series, iPad, Android Phone, and More*.

![Kinmates T01](/img/kinmates.jpg)

The intent was to to use it with my e-reader, a Kobo Clara BW. 

![Kinmates T01](/img/kobo-clara.jpg)

These devices do not work together out of the box; in fact, the marketing material for the Kinmates T01 *specifically* states that it is not compatible with e-readers, only the Kindle app on phones and tablets.

Well, I took that personally.

## Step 1: Nickel Menu

The first step is to install [Nickel Menu](https://pgaskin.net/NickelMenu/) on the Kobo. If you do any kind of Kobo customization, you need this.

## Step 2: Bluetooth Page Turner Support

The next step is to install [Bluetooth page turner for Kobo](https://github.com/tsowell/kobo-btpt) (kobo-btpt) on the Kobo. This little utility listens for Bluetooth input events and creates page turn events.

## Step 3: Event Codes

The hardest part of the process was figuring out the event codes for the Kinmates device. I used the [`evtest` utility](https://cgit.freedesktop.org/evtest/) on my Linux machine to capture the events.

The Kinmates T01 has two different modes: "short video" mode (intended for scrolling TikTok) and "ebook mode" (intended for use with the Kindle mobile app). The round button on the device can be held to switch between the two modes.

### Ebook Mode

Here are the codes I read from `evtest` with the Kinmates T01 in "ebook mode" and the wheel turned *down*:

```
Event: time 1735237106.812312, type 4 (EV_MSC), code 4 (MSC_SCAN), value d0042
Event: time 1735237106.812312, type 1 (EV_KEY), code 330 (BTN_TOUCH), value 1
Event: time 1735237106.812312, type 1 (EV_KEY), code 320 (BTN_TOOL_PEN), value 1
Event: time 1735237106.812312, type 3 (EV_ABS), code 0 (ABS_X), value 3328
Event: time 1735237106.812312, -------------- SYN_REPORT ------------
Event: time 1735237106.840276, type 3 (EV_ABS), code 0 (ABS_X), value 3400
Event: time 1735237106.840276, -------------- SYN_REPORT ------------
Event: time 1735237106.842242, type 3 (EV_ABS), code 0 (ABS_X), value 3100
Event: time 1735237106.842242, -------------- SYN_REPORT ------------
Event: time 1735237106.873265, type 3 (EV_ABS), code 0 (ABS_X), value 2800
Event: time 1735237106.873265, -------------- SYN_REPORT ------------
Event: time 1735237106.902462, type 3 (EV_ABS), code 0 (ABS_X), value 2500
Event: time 1735237106.902462, -------------- SYN_REPORT ------------
Event: time 1735237106.930456, type 3 (EV_ABS), code 0 (ABS_X), value 2200
Event: time 1735237106.930456, -------------- SYN_REPORT ------------
Event: time 1735237106.961263, type 3 (EV_ABS), code 0 (ABS_X), value 1900
Event: time 1735237106.961263, -------------- SYN_REPORT ------------
Event: time 1735237106.993299, type 3 (EV_ABS), code 0 (ABS_X), value 1600
Event: time 1735237106.993299, -------------- SYN_REPORT ------------
Event: time 1735237107.020481, type 3 (EV_ABS), code 0 (ABS_X), value 1300
Event: time 1735237107.020481, -------------- SYN_REPORT ------------
Event: time 1735237107.051456, type 3 (EV_ABS), code 0 (ABS_X), value 1000
Event: time 1735237107.051456, -------------- SYN_REPORT ------------
Event: time 1735237107.082455, type 4 (EV_MSC), code 4 (MSC_SCAN), value d0042
Event: time 1735237107.082455, type 1 (EV_KEY), code 330 (BTN_TOUCH), value 0
Event: time 1735237107.082455, type 1 (EV_KEY), code 320 (BTN_TOOL_PEN), value 0
Event: time 1735237107.082455, -------------- SYN_REPORT ------------
Event: time 1735237107.082465, type 3 (EV_ABS), code 0 (ABS_X), value 700
Event: time 1735237107.082465, -------------- SYN_REPORT ------------
```

If you squint a little, you can see what it's doing: it's simulating a finger swiping across the horizontal axis of the screen, using `BTN_TOUCH` and `BTN_TOOL_PEN` to simulate a touch event and then moving the `ABS_X` axis to simulate the swipe.

Let's see what happens when the wheel is turned *up*:

```
Event: time 1735237152.352476, type 4 (EV_MSC), code 4 (MSC_SCAN), value d0042
Event: time 1735237152.352476, type 1 (EV_KEY), code 330 (BTN_TOUCH), value 1
Event: time 1735237152.352476, type 1 (EV_KEY), code 320 (BTN_TOOL_PEN), value 1
Event: time 1735237152.352476, type 3 (EV_ABS), code 0 (ABS_X), value 512
Event: time 1735237152.352476, -------------- SYN_REPORT ------------
Event: time 1735237152.382446, type 3 (EV_ABS), code 0 (ABS_X), value 650
Event: time 1735237152.382446, -------------- SYN_REPORT ------------
Event: time 1735237152.442452, type 3 (EV_ABS), code 0 (ABS_X), value 948
Event: time 1735237152.442452, -------------- SYN_REPORT ------------
Event: time 1735237152.471928, type 3 (EV_ABS), code 0 (ABS_X), value 1250
Event: time 1735237152.471928, -------------- SYN_REPORT ------------
Event: time 1735237152.559278, type 3 (EV_ABS), code 0 (ABS_X), value 1550
Event: time 1735237152.559278, -------------- SYN_REPORT ------------
Event: time 1735237152.621952, type 3 (EV_ABS), code 0 (ABS_X), value 1850
Event: time 1735237152.621952, -------------- SYN_REPORT ------------
Event: time 1735237152.653442, type 3 (EV_ABS), code 0 (ABS_X), value 2150
Event: time 1735237152.653442, -------------- SYN_REPORT ------------
Event: time 1735237152.679486, type 3 (EV_ABS), code 0 (ABS_X), value 2450
Event: time 1735237152.679486, -------------- SYN_REPORT ------------
Event: time 1735237152.682305, type 3 (EV_ABS), code 0 (ABS_X), value 2750
Event: time 1735237152.682305, -------------- SYN_REPORT ------------
Event: time 1735237152.712306, type 3 (EV_ABS), code 0 (ABS_X), value 3050
Event: time 1735237152.712306, -------------- SYN_REPORT ------------
Event: time 1735237152.712321, type 4 (EV_MSC), code 4 (MSC_SCAN), value d0042
Event: time 1735237152.712321, type 1 (EV_KEY), code 330 (BTN_TOUCH), value 0
Event: time 1735237152.712321, type 1 (EV_KEY), code 320 (BTN_TOOL_PEN), value 0
Event: time 1735237152.712321, -------------- SYN_REPORT ------------
Event: time 1735237152.712324, type 3 (EV_ABS), code 0 (ABS_X), value 3350
Event: time 1735237152.712324, -------------- SYN_REPORT ------------
```

Same thing, but in reverse.

### Video Mode

In video mode, the events are surprisingly similar. Here's the wheel turned *down*:

```
Event: time 1735237199.210268, type 4 (EV_MSC), code 4 (MSC_SCAN), value d0042
Event: time 1735237199.210268, type 1 (EV_KEY), code 330 (BTN_TOUCH), value 1
Event: time 1735237199.210268, type 1 (EV_KEY), code 320 (BTN_TOOL_PEN), value 1
Event: time 1735237199.210268, type 3 (EV_ABS), code 1 (ABS_Y), value 3200
Event: time 1735237199.210268, -------------- SYN_REPORT ------------
Event: time 1735237199.239524, type 3 (EV_ABS), code 1 (ABS_Y), value 2600
Event: time 1735237199.239524, -------------- SYN_REPORT ------------
Event: time 1735237199.242253, type 3 (EV_ABS), code 1 (ABS_Y), value 2300
Event: time 1735237199.242253, -------------- SYN_REPORT ------------
Event: time 1735237199.272592, type 3 (EV_ABS), code 1 (ABS_Y), value 2000
Event: time 1735237199.272592, -------------- SYN_REPORT ------------
Event: time 1735237199.300304, type 3 (EV_ABS), code 1 (ABS_Y), value 1700
Event: time 1735237199.300304, -------------- SYN_REPORT ------------
Event: time 1735237199.329275, type 3 (EV_ABS), code 1 (ABS_Y), value 1400
Event: time 1735237199.329275, -------------- SYN_REPORT ------------
Event: time 1735237199.361268, type 3 (EV_ABS), code 1 (ABS_Y), value 1100
Event: time 1735237199.361268, -------------- SYN_REPORT ------------
Event: time 1735237199.392279, type 3 (EV_ABS), code 1 (ABS_Y), value 800
Event: time 1735237199.392279, -------------- SYN_REPORT ------------
Event: time 1735237199.422465, type 3 (EV_ABS), code 1 (ABS_Y), value 500
Event: time 1735237199.422465, -------------- SYN_REPORT ------------
Event: time 1735237199.450320, type 4 (EV_MSC), code 4 (MSC_SCAN), value d0042
Event: time 1735237199.450320, type 1 (EV_KEY), code 330 (BTN_TOUCH), value 0
Event: time 1735237199.450320, type 1 (EV_KEY), code 320 (BTN_TOOL_PEN), value 0
Event: time 1735237199.450320, -------------- SYN_REPORT ------------
Event: time 1735237199.450328, type 3 (EV_ABS), code 1 (ABS_Y), value 200
Event: time 1735237199.450328, -------------- SYN_REPORT ------------
```

It's effectively the same pattern, but with `ABS_Y` instead of `ABS_X`. It turns out that "ebook mode" and "video mode" just mean "swipe left to right" and "swipe up to down" respectively.

### Mapping the Events

Unfortunately, unlike other devices that have a simple key press for page turn, the Kinmates T01 is sending touch events. To map these to page turns, we need to select a specific value of the axis to trigger the page turn. Thankfully the values sent by the device are predictable and there are a few unique values that only occur when the wheel is turned in a specific direction.

Here are the values I observed:

- `ABS_X`:
  - 3400: only when the wheel is turned down
  - 650: only when the wheel is turned up
- `ABS_Y`: 
  - 800: only when the wheel is turned down
  - 2900: only when the wheel is turned up

## Step 4: The Configuration File

The `kobo-btpt` utility reads a configuration file to map the events to page turns. Here's the configuration file I used, based on the event codes I captured above:

```
prevPage EV_ABS ABS_X 3400
prevPage EV_ABS ABS_Y 800
nextPage EV_ABS ABS_X 650
nextPage EV_ABS ABS_Y 2900
```

This maps "scroll up" and "scroll down" to page turns in both ebook and video modes. I made "up" correspond to "next page" and "down" correspond to "previous page" because it feels more natural to me to rotate the wheel away from my thumb. You might reverse them.

This configuration file is placed in the following location on the Kobo (since the Bluetooth name of this device is T01):

```
.btpt/T01
```

## Step 5: Connecting the Kinmates T01

Hold the round button on the Kinmates T01 to enter pairing mode, then pair it with the Kobo. The device should show up as "T01" in the Bluetooth settings.

## Step 6: Profit

With the Kinmates T01 paired and the configuration file in place, the device should now work as a page turner for the Kobo. Happy reading!

