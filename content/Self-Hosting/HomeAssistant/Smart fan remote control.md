---
created: 2024-07-19T17:31
updated: 2025-07-21T17:15
tags:
  - house
  - homeassistant
  - esp32
  - shareable
---
I moved in to a house that came with 3 "smart" fans and 2 remote controls. Here's the process I went through when evaluating [[#Replacing the remote]] and eventually landing on [[#Using the controller in HomeAssistant]] instead. Now I have a controller in each room so I can remotely manage the fan/light.

# Identifying the remote
The only text on the remote is `FCC ID:2AZ3EKJDGS-001`
- https://fccid.io/2AZ3EKJDGS-001
	- Frequency range: [2.402-2.48 GHz](https://fccid.io/frequency-explorer.php?lower=2402.00000000&upper=2480.00000000)

## Pairing the remote
Let's look at the [manual from fcc.io lookup](https://fcc.report/FCC-ID/2AZ3EKJDGS-001/5270739.pdf):
> Turn on the power switch and press “di” for 3 seconds after hearing the sound “check code” button. The receiver clicks ”tick” twice. Check code successfully. Such as If no code matching is successful, turn off the power and wait for 1 minute to repeat.

I looked for a better explanation because I wasn't sure what "di" was, which lead to [this very similar-looking remote in a manual](https://images.thdstatic.com/catalog/pdfImages/e0/e09c3819-c038-4a87-872b-94a0e389e1fa.pdf) that has a \[Fan off\] in the same location as \[SETUP\].
> Try re-pairing the remote. Power on the fan, when you hear the sound of "beep", please press the \[Fan off\] button until you hear a long beep sound. It means successful.

For our remote, this means:
Power on the fan via wall switch. When you hear the fan beep, hold the \[SETUP\] button on the remote for 3 seconds. You will hear the fan double-beep if it is successful.

(I probably could have guessed that the \[SETUP\] button was involved, but I had been pressing the button instead of holding.)

## Un-pairing the remote
So far I have tried some suggestions found online:
- breaker being shut off >1min
- turning fan on and off 6 times
	- https://old.reddit.com/r/CeilingFans/comments/m3ccyc/reset_fanimation_fan_wifi/

These didn't work consistently which has me confused at the moment.

# Replacing the remote
## Can I buy the remote?
I don't see any available remote-only options, all of the ones that look just like mine are included with a fan.
### Amazon remote
I found a black remote (a little more stylized) with a button layout that is nearly identical to the white remotes I got with the house, but doesn't work as a drop-in replacement.
```
Printed on Board:
XKS-9685A

SOIC:
XL19KF60611
B000333
2412HAGK

antenna:
24.000
MHZ

capacitor (MLCC?):
222
```

## Can I emulate the remote?
This fan uses BLE, so the proprietary 2.4GHz signal research I wrote about here has been moved to [[Emulating proprietary 2.4GHz signals]].

## Can I use an app as the remote?
Is there an app that can already communicate with the fan? **Yes!**

I found `ZhiMeiDengKong` on [a HomeAssistant forum thread](https://community.home-assistant.io/t/controlling-ble-ceiling-light-with-ha/520612/125?page=7), but in my case I needed **FanLamp Pro** ([iOS](https://apps.apple.com/us/app/fanlamp-pro/id1570374552) / [Android](https://play.google.com/store/apps/details?id=com.jingyuan.fan_lamp&hl=en_US&pli=1)) .

I only have an old Nexus 6P available, so I had to download 1.3.9 from APK Pure ([direct download link](https://apkpure.com/fanlamp-pro/com.jingyuan.fan_lamp/downloading/1.3.9)).


## Can I integrate with Home Assistant?
Yes!

**Research**:
- ["Controlling BLE ceiling light with HA" thread on HA forums](https://community.home-assistant.io/t/controlling-ble-ceiling-light-with-ha/520612/8)
	- original solution: [lampify](https://github.com/MasterDevX/lampify)
	- improved and currently-supported solution: [aronsky's ble_adv_controller](https://github.com/aronsky/esphome-components)
	- adventurous new update: [NicoIIT's listen&decode update on HA forums](https://community.home-assistant.io/t/controlling-ble-ceiling-light-with-ha/520612/208)
		- [NicoIIT's ble_adv_controller](https://github.com/NicoIIT/esphome-components/tree/dev)
- (slighty) related: BLE sniffing with [nRF Connect](https://play.google.com/store/apps/details?id=no.nordicsemi.android.mcp&hl=en_US&pli=1)
	- [download releases from GitHub](https://github.com/NordicSemiconductor/Android-nRF-Connect/releases)

### ESPHome setup
https://esphome.io/guides/getting_started_hassio.html

Use this link to add ESPHome to HomeAssistant:
https://my.home-assistant.io/redirect/supervisor_addon/?addon=5c53de3b_esphome&repository_url=https%3A%2F%2Fgithub.com%2Fesphome%2Fhome-assistant-addon

### ble_adv_controller component
https://github.com/aronsky/esphome-components/blob/main/components/ble_adv_controller/README.md

#### Installing controller firmware on Raspberry Pi Pico W
```yaml
# fan.yaml
external_components:
  components: [ ble_adv_controller ]
  source: github://aronsky/esphome-components@main
```

I discovered that `ble_adv_controller` relies on `esp32_ble_tracker`. Unfortunately this BT stack used by `esp32_ble_tracker` specifically requires ESP32 hardware, and there is no plan to rewrite the code at the moment (https://github.com/esphome/issues/issues/3828). The RPi Pico W that I had laying around will not work for this project, but at least I got some ESPHome configuration practice.

#### Installing controller firmware on ESP32C6
I decided to order the [XIAO ESP32C6 from seeed studio](https://www.seeedstudio.com/Seeed-Studio-XIAO-ESP32C6-p-5884.html) which will get me the BLE I need, USB-C port, and Thread/Matter support if I ever need it in the future. It barely costs more than older and less capable ESP32 devices, especially if were to  to pick one up locally at MicroCenter ($25 Spark boards!).

**Guides:**
- [espressif esp32c6 - establish serial connection](https://docs.espressif.com/projects/esp-idf/en/stable/esp32c6/get-started/establish-serial-connection.html#flash-using-usb)
- [espressif esp-idf 5.2 - macos setup](https://docs.espressif.com/projects/esp-idf/en/release-v5.2/esp32/get-started/linux-macos-setup.html)
- [seeed studio xiao esp32c6 - official "getting started" guide](https://wiki.seeedstudio.com/xiao_esp32c6_getting_started/)
- [seeed studio xiao esp32c6 - Adrian Bateman's Home Assistant setup](https://adrianba.net/2024/12/01/esp32c6-and-home-assistant/)
- (slighty) related: [seeed studio xiao esp32c6 - embedded swift setup](https://github.com/Seeed-Studio/wiki-documents/blob/docusaurus-version/docs/Sensor/SeeedStudio_XIAO/SeeedStudio_XIAO_ESP32C6/XIAO_ESP32C6_Embedded_Swift.md)
```shell
# Use ESPHome Builder to create a new device.
# Edit the yaml with the changes shown below, then compile locally
mkdir -p ~/esp/config; cd ~/esp/config; vim fan-controller.yaml
docker run --rm -it -v /home/user:/config ghcr.io/esphome/esphome compile fan-controller.yaml

# install esp-idf to flash firmware file
brew install cmake ninja dfu-util ccache
cd ~/esp; git clone -b release/v5.2 --recursive https://github.com/espressif/esp-idf.git
cd ~/esp/esp-idf; ./install.sh esp32c6

# flash firmware file
cd ~/esp/esp-idf; esptool.py -p /dev/cu.usbmodem101 --before default_reset --after hard_reset --chip esp32c6 write_flash --flash_mode dio --flash_size detect --flash_freq 40m 0x0 ~/esp/config/.esphome/build/fan-controller/.pioenvs/fan-controller/firmware.factory.bin
```

ESPHome device yaml:
```diff
esphome:
  name: fan-controller
  friendly_name: fan-controller
>>>
+  platformio_options:
+    platform: https://github.com/mnowak32/platform-+espressif32.git#boards/seeed_xiao_esp32c6
+
+external_components:
+  - source: github://aronsky/esphome-components@main
+
+ble_adv_controller:
+  - id: fan_controller
+    encoding: fanlamp_pro
+
+light:
+  - platform: ble_adv_controller
+    ble_adv_controller_id: fan_controller
+    name: Main Light
+
+fan:
+  - platform: ble_adv_controller
+    ble_adv_controller_id: fan_controller
+    name: Fan
+
+button:
+  - platform: ble_adv_controller
+    ble_adv_controller_id: fan_controller
+    name: Pair
+    cmd: pair

esp32:
-  board: esp32dev
-  framework:
-    type: arduino
+  board: seeed_xiao_esp32c6
+  flash_size: 4MB
+  variant: esp32c6
+  framework:
+    type: esp-idf
+    version: 5.2.1
+    platform_version: 6.9.0
+    sdkconfig_options:
+      CONFIG_ESPTOOLPY_FLASHSIZE_4MB: y

# Enable logging
logger:
...
```

#### Using the controller in HomeAssistant
In the HomeAssistant, navigate to the fan-controller device settings and set Encoding to `fanlamp_pro - v1`.

Flip physical light switch off. Flip light switch on and immediately press the PAIR button under the fan-controller device settings. Light will blink if pairing was successful.

Now the light and fan are controllable directly from HomeAssistant!



#### Things to address after setting up controller
Right after setup, I ran into an issue where one controller was addressing multilpe fans, but it turned out to be a duplicate MAC address. I cleared build cache and recompiled firmware to get a new MAC address for improperly-configured controller.

Advanced settings:
https://github.com/aronsky/esphome-components/blob/main/components/ble_adv_controller/CUSTOM.md