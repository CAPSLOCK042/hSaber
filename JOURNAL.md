---
title: "hSaber"
author: "James Xiao"
description: "Neopixel saber that has interspersed high-power LEDs to display flashes of light upon contact for effect, can swap out outer casing to be lit on fire."
created_at: "2025-06-20"
---

## June 24: Tested out LEDs in basement
- Found a spare lighting-application LED light strip in basement. Tested it with a DC power supply, got up to pretty bright while drawing around 0.2A (200mA).
- Unsure if it's bright enough for the application - a lot of manual labor will have to be done (cutting, putting on pins for soldering) for the eventual high-power LEDs
- Eventual goal is to get it as bright as possible - only 2 LEDs will be on at a time MAX, so drawing too much current from a battery is not a concern, even though there are many LEDs
- Time spent: **40 mins**
- Next steps: Decide if I want to use these LEDs, or invest in a less-janky solution?

## June 26: Thought about/researched LED control methods
- So driving the low-power LED strips is easy - the pre-programmed ws2812b strips already make individually addressing LEDs easy. But manually controlling single-color, white, high-power LEDs is a bit trickier. LED drivers on the market cannot deliver that much current, as they are mainly for logic level-powered LEDs.
- I briefly looked at Charlieplexing which is a viable way of controlling many LEDs given that only one is on at a time, but they are also unsuited for high-current applications as they rely on GPIO pins having 3 states: HIGH, LOW, and HI-Z. If I were to drive them from a MOSFET, they would be incapable of having these 3 states, and while it's probably possible somehow, I don't think it's worth.
- I also looked at controlling it with MUXes, but MUXes also don't provide enough current :(. Ts tragic
- The option that looks the most promising is a little brute-forcey, but I think I'm just going to get an LED driver and hook each output up to a MOSFET, which will drive the LED. (This could have been done with the MUXes as well, but the driver is easier to implement). This method has been the most obvious in the beginning, but I've strayed away as that would mean buying 30-40 MOSFETs and placing them on the PCB. Perhaps there's a better method, but that's for a more-experienced me. I found some cheap FETs as well that seem to do the trick (logic-level gate threshold voltage, suitable current and voltage draw through source/drain)
- Time spent: **1 hour 30 min**

## July 1: Figured out measurements for saber
- Drew out, measured (not to-scale) diagram of a 2D cross-section of a section of the blade tube.
- Started researching parts online to see if any satisfy these conditions, and what will have to be modified
- Decided on tolerances - this will be a build that is VERY tight on space in one dimension (x-axis), but we can use the y-axis to help us out by stacking a few things
- ![0](https://github.com/user-attachments/assets/43eb91c4-e4f0-4eb8-b12d-e364d25f2806)
- Time spent: **45 min**

## July 2: Did more online research on parts
- Had to make some hard decisions today on what was available on the market. Looked around primarily for PC (polycarbonate) tubing that match my dimensions - found ONE on Amazon that seems to work, so I think I'll go with that
- Also looked for borosilicate glass tubing for when flames might get involved. I shouldn't have done this. I was getting way too ahead of myself.
- ![image](https://github.com/user-attachments/assets/c8bd1b72-fcff-4a28-a316-43389b2b7cf2)
- Time spent: **40 min**

## July 3: Did research on distance sensing for contact with other blades
- The distance sensing is one of the most crucial aspects of the project, as that's what separates it from other neopixel sabers.
- I did a bunch of research and thinking on this before but that was over the course of like a week at interspersed times so I didn't log them. They were mainly during ideation of this project but anyway...I'll just explain it here
- The saber needs to detect when it's in contact with another lightsaber, and more importantly, where that contact occurs. Here are a few options to do that:
  - **Strain gauges**
    - My first idea. By placing strain gauges along the blade, it can detect where it's bending, as contact with the blade will result in a slight bending that the gauges will be able to pick out. However, if the outside material is PC and the inside is reinforced by CF rods, it's hard to imagine that it would bend much. Also, rods don't bend at the point which they are struck, the bending happens all along the space between that point and the support of the rod. This sounds like materials science, physics, and math. Yuck. Also this would rely on me putting a bunch of strain gauges along the side of the tube AND hooking them all up to the MCU, which also sounds yuck.
  - **Piezoelectric sensors**
    - Still doesn't seem like a bad idea, I might go with this along with my 3rd idea (though it has several limitations). Basically, if you place one of these sensors at both ends of the blade, a force/impact will send vibrations along both ends of the rod. By measuring the difference in time from when both of these vibration signals arrive, you can basically tell where on the rod it hit. HOWEVER, they only detect when a HIT is applied - a soft hit might not register. Let's say you hit 2 blades together and then drag them along each other - these sensors don't know that, which is bad. However, these only involve 2 cheap, easy-to-install sensors with some math/programming that doesn't sound too hard, so I might go with it. The MCU needs to have a fast processing speed, so something like a Teensy should be up to the task.
  - **Distance sensors placed along the hilt, looking up towards the tip**
    - This is the best solution out of them all. With 3-4 distance sensors placed near the base of the blade, wrapped around the circumference of the blade and looking up towarsd the tip, we can detect whenever another blade comes in contact with it, as well as where it is. It doesn't require a bunch of sensors laid along the blade. The cons/challenges/limitations are listed below...
- Ok, so I was thinking Ultrasonic sensors for this, as they are the cheapest / I already have one to test. But then I came upon the biggest challenge for this, which is FOV. With a large FOV, the distance sensor would pick up the blade itself instead of what's coming in contact with the blade, which defeats the purpose. Ultrasonic sensors have an incredibly large FOV, so they definitely will pick up the blade itself. I need something with an incredibly small FOV along the length of 1M. Here are the options I found...
  - **Ultrasonic sensor**
    - Yuck, too big FOV, though cheap.
  - **TOF sensor**
    - Most of the cheap, commercial ones also have a pretty big FOV as well. HOWEVER...I found one, that is the VL53L1X, that works well. It has good ranging frequency (50hz), good distance (4m) (4x what I need), and is pretty small in size. However, the default FOV is 27 degrees...BUT IT CAN ADJUST THE FOV DOWN TO 15 DEGREES which sounds good, but is also not enough. Ideally, I need something around 1 to 2 degrees or even less, but that would be hard to find for an affordable price. However, I'm guessing I can put some sort of 3DP tube/baffle in front of the emitter to lower its FOV, ideally something that doesn't reflect the light back. But also, not only can you change the FOV, you can also change the ROI! So, if I make it the smallest FOV, shift it to one side so it doesn't detect the blade, then add my 3DP tube/baffle, it should work...right??
  - **Sharp Distance Sensor**
    - These distance sensors from the company Sharp seem pretty good, are at an affordable price, but have 2 major limitations. The first is the range. There is one that goes from 10 to 80cm, and with my blade at 90cm, it's just short. There's the other that goes from 20cm to 150cm, but 20cm as a minimum distance is too long - I'm gonna have to place it at the bottom of the hilt, in which my hands will get in the way. The second limitation is that there is literally no data on the FOV, or beam width. Some forums say it's 40 degrees, some say it's 20 degrees, some say it's less than 5 degrees. I did find some ppl who have tested it complaining about its high FOV, so I think it's safe to say I won't be using this.
  - **LIDAR Rangefinder (TF-Luna, TFMini-S)**
    - These seem honestly amazing. They use LIDAR, so incredibly narrow FOV (2 degrees). Small form factor, seems perfect...except they're like 23 bucks a piece, cheapest. If I'm ordering 3 of them (minimum), that's like 70 dollars down the drain already. I don't think Hack Club would be very happy with that.
- ...
- Holy cow. That's a lot of stuff to put down. Anyways, my best bet looks like the VL53L1X now. I think I can make do with it.

- Here is an image I guess of some stuff I bookmarked while researching??? idk man
- ![image](https://github.com/user-attachments/assets/30d2b5f1-1e18-418c-a4aa-0c3fd0380b72)
- Total time spent, in addition to writing this: **2 hours**
## July 6: Drew out rough schematic of the system
- This is fun to do because I feel like I'm being productive without actually expending many braincells.
- Anyway, this schematic includes the main MCU (Teensy 4.0), 3 distance sensors (VL53L1X), 2 LED WS2812B light strips, and 2 LED Drivers (TLC5947). These LED Drivers will follow the corresponding schematic from Adafruit
- ![image](https://github.com/user-attachments/assets/b85941b1-2e7f-4615-b39c-35315ec16398)
- A VERY ROUGH outline of the next steps I have:
- ![image](https://github.com/user-attachments/assets/740b0322-260b-4ad5-9404-265060172591)
- Time spent today: **40 min**
## July 9, 10, 11: Worked on Schematic
- Learned the basics of KiCad, how to create schematics, how to create custom symbols, how to create custom footprints, why my downloaded teensy library wasn't allowing me to edit it (it was legacy KiCad)
- I'm planning to have a main board for most the electronics, and a sub board that goes by the hilt to house the 3-4 ToF sensors. A Pmod-type connector will be the bridge between these two
- Designed custom footprints and symbols for VL53L1X
- Added a voltage divider so the Teensy can read the battery input
- Spent a good amount of time sorting through online components for 5V boost converters, inductors, switches, capacitors, etc
- New plan for WS2821 LEDs:
  - Battery is around 4.2-3.6V, but the LED strips need 5 volts
  - A 5V boost converter will be used, but the max I found can only output 1.8A
  - We have 4 meters of LEDs in total - so I will use 4 different 5V boosts - allowing for 1.8A (slightly less) per meter
- Here are some images
- Sub board:
- <img width="550" height="392" alt="image" src="https://github.com/user-attachments/assets/3016eef5-f3af-47fd-94f6-7ae3bf9a82a4" />
- Main board (far from done):
- <img width="1116" height="563" alt="image" src="https://github.com/user-attachments/assets/dd23a87d-4011-4ff3-b52a-782a234b018c" />
- WS2821 Lighting:
-<img width="796" height="722" alt="image" src="https://github.com/user-attachments/assets/af635482-5e9a-4ffe-8c27-47af14830576" />
- Time spent July 9: **1 hr**
- Time spent July 10: **1 hr**
- Time spent July 11: **2.5hr**
