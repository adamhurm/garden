---
author: Adam Hurm
date-created: 2023-05-10
tags:
  - guide
  - 3d_printing
  - shareable
created: 2023-05-10T20:58
updated: 2025-07-21T18:41
---
Install [VSCode](https://code.visualstudio.com/) and install the [PlatformIO extension](https://platformio.org/platformio-ide).

Download the [latest Marlin release](https://github.com/MarlinFirmware/Marlin/releases/) zip and extract the contents.

Open VSCode and select the PlatformIO extension from the Activity Bar.

Select "Open Project" from the PlatformIO Quick Access screen and choose the Marlin folder that you downloaded.

Now edit the `platformio.ini` file. Change the `default_envs` value to match your printer board.
Examples of configurations I have used:
```
default_envs = STM32F103RET6_creality //Ender 3 Pro v4.2.2 board AKA "Ender 3 Pro v1.5"
default_envs = STM32F103RC_btt_USB_maple //BIGTREETECH SKR Mini E3 V2.0
```

Download the [latest Marlin configuration](https://github.com/MarlinFirmware/Configurations/releases) zip and extract the folder that matches your printer board.
Examples of configurations I have used:
```
config/examples/Creality/Ender-3 Pro/CrealityV422/
config/examples/Creality/Ender-3 Pro/BigTreeTech SKR Mini E3 2.0/
```

Move the header (.h) files you just extracted to the `Marlin-X.X.X.X/Marlin` folder.
(Yes, that means overwriting `Configuration.h`, `Configuration_adv.h`, and `Version.h`).

After saving the changes to `platformio.ini` and placing the configuration files in the `Marlin-X.X.X.X/Marlin` folder, we're ready to build:

With PlatformIO project open, Select "Build All" from the Project Tasks listed under the VSCode Side Bar.

If your build succeeds, you should see the path to the created `firmware.bin` file printed above the "SUCCESS" line.

Copy `firmware.bin` to the root of an SD card.

Place the SD card into the printer slot and power cycle the printer to flash the firmware.