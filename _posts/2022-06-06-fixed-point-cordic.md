---
layout: post
title: Fixed Point CORDIC
date: 2022-06-06 22:47:00-0400
description: Architectural design of a fixed point CORDIC algorithm
tags: math dsp
categories: dsp
---

This post was written as a part of my exam preperation for the course in Low Energy and Reconfigurable Systems at AAU.
It is a writeup from a miniproject done in cooperation with two of my fellow students. The miniproject goes through an architectural design of the CORDIC algorithm for calculating the $x$ and $y$ values of a cosine, given an angle. The architecture is at the RTL level and should need no further processing before implementation. Actually it also goes through the mathematical derivation of the CORDIC algorithm, but I won't bore you with that. You just need to know that the algorithm uses "microrotations" to match the angle using the math below

$$x_{i+1} = x_i - \sigma_i \cdot 2^{-i} \cdot y_i$$

$$y_{i+1} = \sigma_i \cdot 2^{-i} \cdot x_i + y_i$$

$$z_{i+1} = z_i - \sigma_i \cdot arctan(2^{-i})$$

Where we set $$x_0 = 1$$, $$y_0 = 0$$ and $$z_0 = \theta$$. Then $$\sigma_i$$ becomes the sign of $$z_i$$. Which will then be the input of the algorithm. It should be noted that the computation $$2^{-i}$$ can be done by shifting the operand we are multiplying it with. Also, we are actually able to calculate $$arctan(2^{-i})$$ beforehand and store it in a table for faster computation.

The pseudo code for the algorithm is listed below.

{% pseudocode %}
Function swap(old, new)
  remaining <- quorumSize
  success <- False
  For Each host
    result[host] <- send(host, propose(old, new))
    If result[host] = "ok"
      remaining--

  If remaining > 1+quorumSize/2
    success <- True

  For Each result
    If success
      send(host, confirm(old, new))
    Else
      send(host, cancel(old, new))
{% endpseudocode %}

The constant $$k$$ is calculated as $$\prod_{i} k_i = \prod_{i} \frac{1}{\sqrt{1+2^{-2i}}}$$. Which means this constant can also be calculated beforehand.

## Numerical aspects (fixed vs. floating point)

We wanted to test some of the aspects related to using a fixed point algorithm. Specifically we wanted to see how many iterations we would expect to have before the algorithm converged over the number of decimal bits used in the fixed point. This was tested in Python using the FixedPoint library. Results are shown in the figure below.

{% include figure.html path="assets/img/posts/Iterations-1.png" class="img-fluid rounded z-depth-1" %}

The figure shows that if a high decimal precision is required, then more iterations are needed before the algorithm converges. Which makes sense. We had an internal discussion whether the results would change if the angles used for the input were generated randomly instead of linearly, so both are included in the figure.
Next, we tested the error between the standard Python floating point precision and the algorithm output over the number of decimal bits. The results of this can be seen in the figure below

{% include figure.html path="assets/img/posts/Error-1.png" class="img-fluid rounded z-depth-1" %}

The figure shows that beyond $$12$$ decimal bits the error is more or less the same. Even $$8$$ decimal bits are useful. Thus based on the previous two figures, we chose a $$12$$-bit 2-complement implementation, with a maximum of $$11$$ iterations before termination. This can be described in the $$Q$$-number format as $$Q3.12$$, where we have four integer bits with MSB being the sign bit.

## Graphical Representations

The hardware implementation of the algorithm could consist of a single CORDIC element, that would then be reused for each iteration. But instead, to take advantage of maximum parallelization and being able to have a continous input, a number of CORDIC elements is used. So basically we are using one CORDIC element per iteration up to the maximum number of iterations. The elements will then be connected to each other sequentially.

### Synchronous Data Flow

Each elements is referenced as $$CE_i$$ and the inputs is connected to $$CE_0$$ while the output comes from $$CE_10$$, as shown in the figure below.

{% include figure.html path="assets/img/posts/ce_chain-1.png" class="img-fluid rounded z-depth-1" %}

We can then open up each element of the previous figure and create a Synchronous Data Flow Graph (SDFG). From the previous figure we note that $$CE_1$$ to $$CE_9$$ operates in the same manner. Taking this into account, we get the following SDFG

{% include figure.html path="assets/img/posts/SDF-1.png" class="img-fluid rounded z-depth-1" zoomable=true%}

Nodes denoted by $$F_i$$ are fork nodes and does not have any arithmatic operation associtated with them. Fork nodes are not considered to cause any execution delays in further system design and is only shown for readability. Nodes that are denoted by $$S_i$$ takes the sign bit from $$z_i$$.

Having created these graphical representations, we can take a deeper dive into the SDFG and create the precedence graph.

### Precedence Graph

The Precedence Graph (PG) also often referred to as a Dependency Graph, is used to visualize dependency between operations or tasks in the algorithm. In doing so, we are also able to determine the Critical Path (currently assuming that all operations require the same amount of time to execute). The PG can be seen in the figure below.

{% include figure.html path="assets/img/posts/PG_time-1.png" class="img-fluid rounded z-depth-1"%}

In order to optimize the architecture and taking advantage of the some parallelism of the system, we can insert delay between all the $$CE$$ elements. By doing so, we make sure that the algorithm can run continously and that all the elements can execute at the same time (still under the assumption that all elements have the same processing time from input to output). As shown in the figure below.

{% include figure.html path="assets/img/posts/ce_chain_delayed-1.png" class="img-fluid rounded z-depth-1"%}

We've also put a limit on the amount of function units that are available for multipleication and addition in each of the $$CE$$'s. Every $$CE$$ will have one shifter (for multiplication) and one adder available. Meaning we end up with 11 shifters, 11 adders, and two multipliers for the 11 iteration CORDIC algorithm. The following scheduling section will show how this is possible.

## Scheduling

