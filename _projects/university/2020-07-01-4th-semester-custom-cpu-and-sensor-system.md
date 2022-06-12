---
layout: page
importance: 4
title: 'Custom CPU and Sensor System'
img: /assets/img/projects/micro_architecture.jpg
category: university
---

Looking back, my fourth semester project was kind of a stretch in terms of useability. We have to create something that we think is exciting, but also fullfills the study regulations. Sometimes, very funky projects is the outcome of that.

A little background before telling you about the project. AAU has multiple student run organization that builds different stuff or simply meet up to discuss different topics. I was very inspired by AAUSAT, which is a project to launch nanosatelittes build by students. Also AAU Racing, that builds and races a Formula type car. However I wanted a project that was grounded and actually had a posibillity to make a scientific and environmental contribution. Since AAUSAT was launching stuff into space and AAU Racing was still using fossil fuels, none of them intrigued me enough to join.

So I had to start my own project. This was where I came up with the idea of a vertical farming system on AAU. The project was to be run by students and would include a vertical farm in a 40 foot shipping container on campus. I contacted my semester coordinator to ask him how I should proceed.

First I was told to make a presentation and a rough budget. Then he got me a meeting with the head of the institute. He liked the idea. They then got me a meeting with a mega-project coordinator and the pro-dean of the university. They also really liked the idea and it actually got approved for funding. Great, then COVID came. Everyone got sent home for the better part of a year and the project got hung up in bureacracy. So the project never got if it’s feet. But I am still glad that I did it and I proved a lot to myself about what is possible as long as you step up. It really showed me that if you want something, you have to work for it and actually ask for what you want.

Anyway, back to the project.

So I had this massive project. For the semester project, all of the adminstration mentioned above thought it would be great if it could be used to get some other students involved. Of course I agreed with that. So I made a project proposal of doing a somewhat scaled version of the system. This scaled version was to be placed in the institute canteen for people to see and ask questions about. My project group and I made a 3D model of the system that you can see below.

{% include figure.html path="assets/img/projects/Konstruktionsbillede1.png" class="img-fluid rounded z-depth-1" zoomable=true %}
The system was designed to be an NFT system, where water runs down through the pipes into a resovoir at the bottom. The water would then be pumped back up and recirculated. It would be placed on an EUR pallet for easy transportation.

Now since the topic of the semester was Digital Design, we couldn’t just make a simpel Arduino control of the system. We had courses on Real-Time Systems and Design of Digital Systems using PSoC’s and FPGA’s. So we decided that we would make two seperate systems. One main unit that would be implemented on an FPGA (basically a CPU) and a another system that would handle all the interfaces to the different sensors. The latter would be implemented on a Cypress PSoC. The two systems would then communicate using a UART interface that was already implemented on the PSoC, but would be a custom implementation in the FPGA.

The different sensors and simpel system diagram can be seen in the figure below.

{% include figure.html path="assets/img/projects/Prototype.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
The following sensors and actuators were used in the system:

- pH-sensor
- EC-sensor
- Water temperature sensor
- Water sensor
- Air conditioning sensor
- Light sensor
- LED Grow lights
- Circulation pump
- Water Pump
- Ventilation Fan

All sensors and actuators were controlled by the PSoC.

Since the semester was focused around digital design, we spent a lot of report pages explaining how different interfaces such as I2C, SPI and UART works. But we also got to dive deeper in how ADC’s work and the different architectures used in most IC’s today.

We made a Real-Time system analysis using CODARTS, which was probably a little bit overkill think back at it. The system did not really need to work real-time and we could have easily left it out. However it was a part of the course we were taught, so we included it to show that we had understood it.

We chose to run a FreeRTOS kernel in the PSoC. It was fun to implement and it also meant that we would have the Real-Time aspect of the curricilum covered.

The FPGA part was implemented on a Terrasic DE0-CV and was designed from a microcontroller standpoint. We used the AVR architecture as a reference point. This meant using the RISC ISA and designing our own instructions, which was very interesting. We ended up implementing instructions for SRAM, ALU, Branch Control and Perifiral Control. The architechture we implemented can be seen below.

{% include figure.html path="assets/img/projects/micro_architecture.jpg" class="img-fluid rounded z-depth-1" zoomable=true %}
The outcome of the project was that we were able to read data of all sensors into the real-time system and transfer them between the CPU and PSoC. We were also able to send new instruction from the CPU to the PSoC.

Because of COVID we never got to see it implemented in the system it was intended to. However we still learned a lot and I consider it one of the most interesting semesters on the Bachelors.