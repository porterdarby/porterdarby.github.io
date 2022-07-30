---
layout: post
title: Magnetic Bed part 2
tags: 3dprinting
---

So I recently [attempted to use a magnetic bed](2022-07-23-Ender-3-v2-with-a-Magnetic-Bed.md) on my Ender 3 v2. Spoiler alert, it didn't work so great. The print bed had some real issues when I was initially using it, mostly adhesion. Because of that, I switch back to the stock glass-plus bed that was originally included in the kit.

I was recently able to update my printer with a little more "oomph" -- I got a [CR Touch](https://store.creality.com/products/cr-touch-auto-leveling-sensor-kit) kit. I thought that it would be a useful tool to have at my disposal, especially since I didn't want to spend like 20 minutes at least once a week re-leveling my printer because I had to move it/make an adjustment.

Thankfully, the installation went (mostly) fine. The end result was that I had the CR Touch installed, and it was able to level against any bed.

Which means that I could re-try using the magnetic bed.

---

The process of switching over to the magnetic bed with the CR Touch proved to be a lot of trial and error. The end result was fiddling with the settings a bunch once everything was installed and using OctoPrint's [Bed Visualizer plugin](https://plugins.octoprint.org/plugins/bedlevelvisualizer/) to help hone in on the relationship between my bed and the CR Touch.

Once that was done, I spent some time working on the Z Offset. This basically tells the printer how much of a difference there is between the CR Touch's trigger point and the nozzle. This allowed me to hone in the Z-axis very well. After a couple of 20mm calibration cube tests, I was able to zero in on my specific parameters.

Since then, I've been working on testing prints on a variety of settings. I've found that the printer for the most part works perfectly, but it seems there might be one small section on the bed that doesn't want to adhere well? I might just be printing too far away (z-offset and/or auto-leveling being weird?), but it's not affecting the overall quality of the build.