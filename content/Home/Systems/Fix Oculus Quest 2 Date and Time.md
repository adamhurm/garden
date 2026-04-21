---
author: Adam Hurm
created: 2022-03-05T00:00
tags:
  - VR
  - software
  - Android
---

## The Problem

The Quest 2 doesn't currently tell you if it encounters issues communicating with [NTP (Network Time Protocol)](https://en.wikipedia.org/wiki/Network_Time_Protocol) servers. If you're having trouble connecting to Wi-Fi networks and getting a "Limited Connection" error message, your Oculus Quest 2 clock is probably out of sync. The date and time may be inaccurate or your timezone could be wrong. The most common scenario is the date being set to some time in 2037.

Unfortunately there is no way to change the date and time through the user interface on the Quest 2. So if your clock falls out of sync, you can't connect to Wi-Fi. If you can't connect to Wi-Fi, then you can't reach NTP servers to update your clock. So your clock will be out of sync until you follow the official support recommendation: factory reset your Quest 2. This is a bit of a nuclear option, so here's another way to accomplish this without having to wipe your device.


## The Fix

[Install SideQuest VR](https://sidequestvr.com/setup-howto) by following the official setup guide.

Install [adb](https://developer.android.com/studio/command-line/adb) on your machine:
- Windows [chocolatey](https://chocolatey.org/install) — `choco install adb`
- macOS [Homebrew](https://brew.sh) — `brew install android-platform-tools`
- Linux — download from the [Android Developers page](https://developer.android.com/studio/releases/platform-tools)

Run the following command to open the hidden Settings app where you can select Date & Time:
```
adb shell am start -a android.intent.action.VIEW -d com.oculus.tv -e uri com.android.settings/.DevelopmentSettings com.oculus.vrshell/.MainActivity
```

Temporarily turn off network time, manually enter your date and time, then turn network time back on. You should now be able to connect to Wi-Fi networks again!

Additionally, you can sideload the [Open Settings](https://sidequestvr.com/app/6744/open-settings) app for the future so you won't have to open a terminal.
