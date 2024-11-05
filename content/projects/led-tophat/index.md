---
title: "LED Tophat"
date: 2023-07-21T19:22:39-05:00
draft: true
tags: []
icon: "/images/projects/led-tophat/icon.gif"
---

{{< video src="/video/projects/led-tophat/cover.mp4" controls="false" >}}

# Introduction

If you've not already found out from the other projects on this site, I am involved with a competition called Odyssey of the Mind. I won't take the time to explain the competition as a whole, but it will suffice to say that peculiar headwear is a common theme amongst the competition's volunteers and participants. Traditionally, most people at the competition wear hats that are easily aqcuired and cheap (floridians like wearing flamingo hats, for instance). Since I have an affinity for things that light up, and cannot say no to overcomplicating things, I decided to make a tophat out of LED matrices.

# The Vision

Briefly, for context, I first embarked on this project in 2019, ultimately creating a rather lackluster first prototype. A few years later, I revisited the project, creating a much better working version that more closely resembles the final product. Using what I learned from the first two prototypes, I laid out some broad design requirements. My goals were for the hat to:

* Have the entire system be self contained, including batteries and power electronics. It must not physically connect to anything outside of the hat.
* Display identifiable content (i.e., have a decently high resolution)
* Be flexible and easy to control
* Be comfortable enough to wear for at least an hour
* Pack nicely into a suitcase and be easy to assemble/disassemble
* Have some kind of interactivity (ideal, but not required)

Ultimately, these requirements proved rather difficult to balance, but I hope to show that the final product adheres to these goals.

# The Displays 

{{< figure src="/images/projects/led-tophat/ws2812bmatrix.jpg" caption="A WS2812B matrix being driven by a Raspberry Pi Zero W (originally used in the first prototype)">}}

When I first prototyped this project, I decided to use WS2812B matrices; they are common and easy to interface with. However, these matrices came with a few drawbacks. First, each LED is on the matrix is always on; for one 8x32 matrix, the power draw could exceed 13 amps, and I needed multiple panels! Second, the pixels are large and not densely packed, making creating an effective display nearly impossible.

Luckily, there exists another type of panel that use multiplexing, smaller (and cheaper) leds, and pack higher pixel densities at a reasonable cost. Explaining multiplexing is out of the scope for this writeup. All you need to know is that instead of illuminating the panels all at once, only a couple rows of each panel are illuminated simultaneous, vastly reducing power draw per LED. I'm not sure exactly what these panels are actually called, though they're typically identified with P# (where # is the millimeter pitch between LEDs).

{{< video src="/video/projects/led-tophat/p4panels.mp4" caption="Two P4 matrices displaying a game of Sonic the Hedgehog" controls="false" >}}

# Driving The Displays

Commonly, these multiplexed displays are driven using an FPGA (e.g., a Colorlight card). Since the displays are multiplexed, they essentially need a constant stream of data with relatively tight timings. Fortunately, an [excellent library](https://github.com/hzeller/rpi-rgb-led-matrix) exists for driving these panels with a Raspberry Pi. Ostensibly, using a Pi was probably not the best choice for the task, but it did make satisfying some of my design goals (ease of control and interactivity) much easier. Importantly, whenever the Pi has to spend time doing things other than driving the display, the display noticably dims. This manifesets itself as an annoying intermittent flicker.

Naturally, freeing up more time for the Pi to do non-display work was helpful. Luckily, by configuring the panels in a specific manner, we can drive more pixels in less time. Specifically, the panels are arranged in "chains", where each panel connects to the panel before it. I could have arranged the six panels on the hat into one chain, but that would have been slower. Instead, arranging the six panels into three chains of two panels roughly doubled the refresh rate I could achieve. Then, by limiting the refresh rate, I can steal that time back for other programs, reducing . However, all of these chains must be connected to the Pi, which leads us to...

# The Adapter PCB

In order to connect all of the panels to the Pi, the inputs for each chain must connect to the GPIO pins on the Pi. Also, since the panels operate on 5V, and the Pi's logic level is 3.3V, so level shifting chips are needed. Luckily, the [excellent library](https://github.com/hzeller/rpi-rgb-led-matrix) had KiCad projects for a HAT (hardware on top of the Pi, not headgear) that performed the connecting and level shifting. I will mostly credit [Henner Zeller](https://github.com/hzeller) for doing all of the hard work, but I did remake the PCB so that it fit to the Pi's footprint, rather than extend off of it.

{{< figure src="/images/projects/led-tophat/pcb.jpg" caption="A picture of my modified driver board">}}

# Power Delivery

After measuring the worse-case power draw of one panel (displaying full white), I determined that I needed around 110 watts of power budget. Originally, I was considering using drill batteries due to their ability to supply high currents. Though, most drill batteries lack protection circuitry and can easily be overdischarged. As it turns out, (good) modern USB-C power delivery power banks can deliver 100 watts, if you ask nicely (by using a trigger module). Additionally, power banks generally come with protections governing power draw, temperature, etc. Of course, using a power bank also meant that the battery could be used flexibly—the battery could be repurposed for everyday use when not on the hat.

The power bank I wound up using could do 100 watts on its USB-C ports, with some extra for a USB-A, which I used to power the Pi. However, USB-C PD power banks deliver 100 watts at 20 volts—I needed 5. This necessitated a buck converter; due to size and weight constraints, I had to settle for a 75 watt converter. While the converter constrained the power available to the panels, it was easy to work around by marginally reducing the panel's brightness.

Ideally, there should also be large capacitors near each panel's power input to help smooth out sudden changes in power demand, but I forwent them since the displays worked fine without them.

# Mechanical Assembly

{{< figure src="/images/projects/led-tophat/cadmodel.jpg" caption="The skeleton of the hat">}}

For the mounting hardware, I decided to CAD and 3D print it myself. I completely abused the sheet metal tools in Fusion so that I could create the mount as a flat surface then curve it to the radius I needed; I would strongly consider a different method in the future since this method makes it practically impossible to CAD on the mounts once they're bent. Nevertheless, the design successfully shapes the panels into a ring. The panels themselves are strong enough to support the vertical loads needed in this project, so the skeleton did not need additional vertical support. Essentially, three panels are wrapped around two rings and secured in place, forming a section of the display. The rings also have captive nuts (i.e., nuts stuck inside the print) such that multiple sections can be easily screwed together. Additionally, a joiner piece connects each panel together along its vertical edges, closing the gap between adjacent panels.

When I was working on the skeleton, I did not have enough panels to make a full assembly, so I had to order more. Annoyingly, the panels I received had a different mounting pattern, so the rings had to be redesigned to accomodate the mounting patterns of panels from two different batches, which is why there are plenty of mounting holes.

{{< figure src="/images/projects/led-tophat/mountingplate.jpg" caption="The wooden mounting plate">}}

On the top section of the display, a laser-cut mounting bracket holds all of the electronics. The placement the battery, buck converter, and Pi had to be carefully considered to avoid issues with wire length, wire clearance, and assembly. Depending on how easily each electrical part needed to be removed, friction fits, screws, and velcro were used as securing methods.

# Software

By the time I got to the software for the hat, I was running out of time; I built this during a busy college semester and the competition was rapidly approaching. Luckily, the [excellent library](https://github.com/hzeller/rpi-rgb-led-matrix) also provided utilities for displaying images, videos, and text to the panels. This made most of the work relatively easy; I wrote simple webserver using flask that could be used to call the utility programs. By having the Pi connect to my phone's personal hotspot, I could connect to the webserver on my phone and control it as if it were an app. I wound up writing code to facilitate efficiently uploading content to the hat; I wanted to be able to quickly show new things on the hat on short notice.

{{< figure src="/images/projects/led-tophat/webinterface.jpg" caption="A screenshot of the web interface (file uploads not shown)">}}

The system configuration was not terribly exotic otherwise; the Pi boots, connects to my hotspot, and launches the webserver.

For fun, and to satisfy my interactivity goal, I decided to run the Pi with a full desktop environment. Again, this worsened display driving performance, but it was acceptable. By running a desktop environment, I was able to use Adafruit's [rpi-fb-matrix](https://github.com/adafruit/rpi-fb-matrix) project to display the desktop on the LED panels. Ultimately (after much compiling), I was able to get Sonic Mania working relatively well. Bet you haven't seen a hat that can play Sonic...

{{< figure src="/images/projects/led-tophat/.jpg" caption="The hat playing Sonic Mania">}}

# Takeaways

{{< figure src="/images/projects/led-tophat/.jpg" caption="The finished product">}}

All of my projects generally contribute to a larger learning experience, and this hat is no exception. The amount of CAD involved increased with each prototype, and my proficiency with Fusion also increased accordingly. Between the second prototype and final version, I learned to better emphasize design for manufacturing and assembly; the captive nuts saved so much hassle with assembly. The wooden mounting plate gave me plenty of chances to get good at using laser cutters, particularly for multi-pass operations. Additionally, the adapter board was the first I made PCB outside of a class, which was rather satisfying. Overall, I'd say this project was a great exercise in utilizing different design and manufacturing techniques. That is all to say, I had quite a lot of fun building this hat, and I'm quite satisfied with the project.

I do have a few ideas for improvements. Notably, the hat only really fits large craniums (like mine). Having some kind of adjustment mechanism and padding would be the finishing touch, I think. Though, I doubt I'll act on this idea soon; I'm happy to call the project finished here. I can only spend so much time on this thing, after all...

# Bonus Stuff

Since you stayed for the whole story, here's some bonus photos:

{{< figure src="/images/projects/led-tophat/bonus-firsthat.jpg" caption="The very first prototype using a WS2812B matrix and a Raspberry Pi Zero W" >}}

{{< figure src="/images/projects/led-tophat/v2-mountingbrackets.jpg" caption="The mounting brackets for the second prototype, which only used four panels. These were basically impossible to assemble due to screw clearances and the lack of captive nuts." >}}

{{< figure src="/images/projects/led-tophat/v2-electronics.jpg" caption="The electronics for the second prototype. A 3D printed mounting plate (not shown here) was used in this design, but it wasn't very good. Also, this prototype used a pre-made Adafruit driver board, which I ditched in the final version." >}}


Thanks for reading!