---
created: 2024-01-16T17:49
updated: 2025-07-21T18:27
tags:
  - homelab
---
If you move in to a house with AT&T Fiber, you will likely receive a BGW320-500 ([FCC report pdf](https://fcc.report/FCC-ID/O6ZBGW320/4522478.pdf)) or some similar model.

You can use the **IP Passthrough** feature to relegate the BG320-500 to modem-only responsibilities and pass all routing responsibilities on to your router. If you change your router at any point in the future, follow Steps 11-14 to update the entry with the MAC address of the new router.

---

All credit to [dimoadiamond on Unifi community forums](https://community.ui.com/questions/BGW320-500-Bridge-Mode-and-or-IP-Passthrough-Question/99786f13-1f76-46dd-9801-7102fd1d44d7?page=2#answer/977c2d10-c6ac-408e-af75-005466a19985) for this solution:

1.  Login to the BGW210-700's web-based configuration interface in your web browser using the link:  [https://192.168.1.254](https://192.168.1.254/)

2. Go to the "Home Network" tab -> "Wi-Fi" tab

3. Set Home SSID Enable to "Off"

4. Set Guest SSID Enable to "Off"

5. Set 2.4 GHz Wi-Fi operation to "Off"

6. Set 5 GHz Wi-Fi operation to "Off"

7. Go to the "Firewall" -> "Packet Filter" tab. Click on the "Disable Packet Filters" button.

8. Go to the "Firewall" -> "IP Passthrough" tab. Select "Passthrough" in the "Allocation Mode" option

9. Do not enter anything for the "Default Server Internal Address". Leave this field blank

10. In the "Passthrough Mode" selection choose "DHCPS-Fixed".

11. Type in the RBR50 MAC address for your router under "Manual Entry". The MAC address should be in traditional hexadecimal format xx:xx:xx:xx:xx:xx where x's should be values from 0-9 or letters a-f, separated by colons.

12. The Passthrough DHCP Lease value defaults to 10 minutes. You cannot change this.

13. Click "Save" at the bottom. Go back and check to make sure that the changes were implemented. If not, do it again.

14. Click on the "Device" tab and select "Restart Device". It will take a couple of minutes.
