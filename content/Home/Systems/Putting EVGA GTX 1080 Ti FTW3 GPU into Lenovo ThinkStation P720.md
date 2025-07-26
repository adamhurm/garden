---
created: 2025-07-07T00:49
updated: 2025-07-26T12:25
tags:
  - homelab
  - thinkstation
---
I bought a Lenovo P720 on GovDeals and I want to put my EVGA GTX 1080 Ti FTW3 GPU in it.

## P720 Manuals / Guides:
- [User Guide](https://download.lenovo.com/pccbbs/thinkcentre_pdf/p720_ug_en.pdf) (pdf)
- [Spec Sheet](https://psref.lenovo.com/syspool/Sys/PDF/ThinkStation/ThinkStation_P720/ThinkStation_P720_Spec.pdf) (pdf)
- [Hardware Maintenance Manual](https://download.lenovo.com/pccbbs/thinkcentre_pdf/p720_hmm.pdf) (pdf)
- [Power Configurator guide](https://download.lenovo.com/pccbbs/thinkcentre_pdf/p920_p720_power_configurator_v1.6.pdf) (pdf)
- [Removal and Replacement Videos](https://pcsupport.lenovo.com/us/en/products/workstations/thinkstation-p-series-workstations/thinkstation-p720/30ba/solutions/HT516264)

I confirmed that the ThinkStation P720 and GTX 1080 Ti are *generally* compatible, which gave me the confidence to try this out:
- [1080Ti compatibility thread on Lenovo forums](https://forums.lenovo.com/t5/ThinkStation-Workstations/GTX-1080-Ti-compatibility-with-Thinkstation-workstations/m-p/3698569)
## Starting point: I need another GPU power supply cable
There are two ports on the motherboard that deliver GPU power: GFX_PWR1 and GFX_PWR2.

GFX_PWR1 came with a cable already plugged in: (`Lenovo FRU p/n:00XL280`)
![[Lenovo power cable FRU 00XL280.jpg]]

My GPU (the EVGA GTX 1080 Ti FTW3) requires a 8+6pin cable for power, so I will need a second cable to deliver power from GFX_PWR2.

A quick search on eBay and tech repair sites shows that this one `OOXL280` cable costs $40-60 which I would love to avoid paying if possible.

## Troubleshooting steps
### Is there a cheaper fix?

#### Bad idea: Buy the wrong adapter
I bought this F6pin-M8pin adapter on Amazon before I took the computer case apart enough to see the original cable (pictured above). This one was a `00XL159` that I saw listed in ThinkStation P720 docs. That was a bad idea because I actually needed M-M. Whoops.
![[StarTech 6pin to 8pin (front).jpg]]
![[StarTech 6pin to 8pin (back).jpg]]
#### Better idea: Use the right adapter from the wrong manufacturer
I had a M6pin-M6+2pin cable which fit the general requirements here, but it originally came with a 1000W PSU (EVGA 1000G+). I was unable to power the 1080Ti with it. The 1080Ti would have green LEDs and one blinking red LED, which indicates a power delivery issue. Bad cable?

### Investigating TDP concerns

#### What power supply is in the P720? 
The ThinkStation P720 comes with one of [690W / 900W / 1000W PSU](https://download.lenovo.com/pccbbs/thinkcentre_pdf/thinkstation_rtx_gpu_support_matrix_v1.0.pdf).

Mine came with a 690W PSU:
![[ThinkStation P720 690W PSU.jpg]]

#### Why is the 690W PSU not capable enough?
I would have thought 690W would be capable of driving the 1080Ti GPU. According to the [P720 Processors and Graphics Power Usage doc ](https://psref.lenovo.com/syspool/Sys/PDF/ThinkStation/ThinkStation_P720/ThinkStation_P720_Spec.pdf) , it’s up to 165W for the Xeon Silver 4114 CPU and 50W for the [NVIDIA Quadra P1000 GPU](https://www.nvidia.com/content/dam/en-zz/Solutions/design-visualization/productspage/quadro/quadro-desktop/quadro-pascal-p1000-data-sheet-us-nv-704475-r1.pdf). Should leave plenty of TDP headroom my GPU (280W for mine, vs 250W of the base model 1080Ti), so I think that I just really need the official PSU cable or I need to enable the second GPU power port on the motherboard via BIOS.

### Is there a BIOS setting controlling the motherboard power ports?
I got a miniDP cable so that I can get video out of the NVIDIA Quadro P1000 GPU. I plugged in a keyboard, started up the computer, and mashed **F1** to get into BIOS settings. I didn’t see any obvious options in BIOS settings, so I think I need the proper OEM M6pin-M6+2pin cable.


## (Partial) Solution: Buy official GPU power cable from Lenovo
Go to [Lenovo parts lookup](https://support.lenovo.com/us/en/parts-lookup) and search by **Part Number**: `00XL280`
![[Lenovo GPU cable available for purchase.png]]

Really pleasant experience with Lenovo support, because the part was 50% off and free shipping (2 days in my case).

Unfortunately, even when using this cable, I still see the blinking red light on the GTX1080Ti: 
![[!assets/1080Ti led green =).jpg]]
![[1080Ti led red =(.jpg]]

It really seems like the 690W PSU does not deliver power to the second GFX power port on the motherboard. This cable was still needed, but I will also probably need to either upgrade the official power supply or add a second power supply specifically for my GPU. I would prefer to go the official route to minimize jank in my setup.

## Solution: Buy official 1000W power supply from Lenovo
I decided to thoroughly consult the ThinkStation P720 User Guide before making any more moves and sure enough I found this critically important detail: "*Note: For computers with 690-watt PSU, only the 6-pin power connector 1 is active.*" That explains why the GFX_PWR2 port wasn't getting any power in the first place.

Unfortunately buying a new 900W PSU from Lenovo costs much more than the GFX power cable. In fact, it costs even more than the $190 I paid for the P720 in the first place. I don't even see any 1000W PSUs listed on this support site.
![[ThinkStation P720 PSU vs GFX PWR cable.png]]

I looked on eBay and found the [900W PSU for \$80](https://www.ebay.com/itm/226127185307) and the [1000W PSU for \$108](https://www.ebay.com/itm/236153586440). Based on the [P720 Power Configurator guide](https://download.lenovo.com/pccbbs/thinkcentre_pdf/p920_p720_power_configurator_v1.6.pdf), the 900W PSU can drive a 250W graphics card and the 1000W PSU can drive a 300W graphics card. I'm worried the 900W PSU wouldn't work out with my GPU that hits around ~280W, so I'll accept the ~$30 price difference so that I don't have to troubleshoot more. I went ahead and pulled the trigger on the 1000W PSU. I want to follow the "officially-supported" path even though the P720 has reached End of Development support.

Plugged in the 1000W PSU and tried again. Success!
![[1080Ti led blue =).jpg]]