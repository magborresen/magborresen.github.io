---
layout: page
importance: 6
title: 'Arduino Plotter (EMC probe holder)'
category: university
---

On the 2nd semester of my studies I was in a project group with 5 other students. Now we had dipped our toes in the water from the 1st semester and we had some more interesting project proposals. My group chose to go with a proposal for automating probe moves in an EMI/EMC testing chamber.

The project was done in collaboration with FORCE Technology, who does EMC/EMI testing for other companies. When doing this sort of testing, you have to make sure that you are shooting the right amount of interfering signals on to a grid on which the DUT is located. This requires probing in specific locations in the grid to make sure that the radiating interference in uniform across the grid. We visitied FORCE at their location in Aarhus to get a better feel of what was actually needed. A second group was also working on the same project case and came up with a very different we solution.

We proposed a solution which we called the 2D spider probe. Because the probe would be suspended from two fishing lines, hence the spider part and it moved in 2D.

{% include figure.html path="assets/img/spider-probe.png" class="img-fluid rounded z-depth-1" zoomable=true %}
The probe is controlled by an Arduino Mega with a custom shield on it that we made in KiCad and got CNC’ed on a PCB machine at the university. A pully system and two stepper motors moves the probe to the positions outlined by the intersecting strings. The wooden cross-hair coming from the buttom of the frame is the calibration point.

The final product wasn’t as focused on electronics as we would have liked it to be. However we had fun with it at came out with a decent result.