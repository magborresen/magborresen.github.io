---
layout: page
importance: 1
title: 'Track-Before-Detect FMCW Radar Algorithm Exploration'
img: /assets/img/projects/2nd-semester-graduate/likelihood_map_3d-1.png
category: university
---

Now that there's only 3 semesters left of my university career, we wanted to try something new. I have never worked with radio systems much and particularly not radar. Thus when the oppertunity to do a project within Numerical Scientific Computing, my prosective group member and I went to one of the professors in the communications department and asked if he had a project that we could work on. He suggested we could work on a target tracking radar system utilizing a track-before-detect (TBD) algorithm. He actually presented a paper that he had cowritten showcasing a radar signal model, target state-space model and an algorith called a Particle Filter. Needless to say, we accepted his proposal and started researching. The goal of the project would be to create a simulation model that could be used to evaluate the computational complexity of such a tracking system.

Since we knew absolutely nothing about radar systems we took to the internet and some recommended books from our supervisor. Ofcourse we commited the usual crime of throwing ourselves over the actual algorithm before we had gotten an overview of what different kinds of radar systems was available. However we know that we wanted to see if we could simulate a Multiple-Input Multiple-Output (MIMO) system using the signal model from the article our supervisor had cowritten, by the way, the article can be found [here](https://www.researchgate.net/publication/224386509_A_Single-Stage_Target_Tracking_Algorithm_for_Multistatic_DVB-T_Passive_Radar_Systems).

So let's see what kind of usecase we can create for this radar system.

{% include figure.html path="assets/img/projects/2nd-semester-graduate/usecase-1.png" class="img-fluid rounded z-depth-1" %}

We want to have an array of transmitter antennas and receiver antennas. The transmitters will send out a signal that will bounce of any present targets, but also other objects which we will call clutter. Since we don't have any actual antennas, we need to simulate how a signal would propagate from one antenna, to a target, to another antenna. How should the antennas even be placed in relation to each other? (Note: we will not be taking full use of the advantages that a MIMO system brings to the table because of the limited amount of time a semester offers.)

Next, any radar system needs a signal processing box, because otherwise we would just be receiving weird samples from an antenna. So our code will be focused mostly around this signal processing box.

After the signal processing box we need to use some hardware that contains some more power for the heavy lifting. By that I mean the estimation algorithm. Since we have chosen to implement a particle filter, the same calculations must be run when updating each particle in each observation.

# Radars

Okay, let's first talk a little bit about how radars actually work before diving deeper into the topic of implementing and testing the algorithms. Radar is an old acronym for RAdio Detection And Ranging. Meaning we use electromagnetic radio signals to determine if there is something out there, and in the case there is, how far away it is from the observation point. When emitting a radio wave it will travel in the direction of the emmitance and reach an object. Depending of the Radar Cross Sectional (RCS) area of the object, some energy will be reflected back toward our receiver. The RCS can be a very tricky thing to calculate and it depends on a lot of things. For bladed aircrafts such as commercial drones, it can even depend on the position of the rotors. The military uses special paint on their stealth aircrafts in order to lower the RCS. The recieved power from a reflected wave can be expressed by the radar range equation as shown below

$$P_r = \frac{P_t G_t G_r \lambda^2 \sigma}{(4\pi)^2 R^4 L_a(R)}$$

Where $$P_t$$ is the transmitted power, $$G_t, G_r$$ is gains of the transmitter and receiver, $$\lambda$$ is the wavelength of the carrier, $$\sigma$$ is the RCS, $$R$$ is the distance and $$L_a(R)$$ is an attenuation factor determine by wavelength, media and altitude.

So we can quickly see from the equation that the received signal strength will decrease very rapidly the further the target is from the observation point. In our implementation we've assumed, so far, that antennas are isotropic (even the they are wildly directional) and that the gains are optimal at $$1$$. Thus we can use the to determine the maximum distance that we can detect and track a target from

$$R_{max} = \sqrt[4]{\dfrac{P_t G_t G_r \lambda^2 \sigma}{(4\pi)^3 S_{min} L_a(R)}}$$

Where $$S_{min}$$ is the minimum detectable signal power before the signal is just lost in noise in the receiver. There are also influence from the earths surface and curvature that can be implemented into the mathematical model. For example the signal can hit the ground and be reflected from there onto the target and back to the receiver. This would increase the observed delay between transmission and reception. However we have not considered that for this project.