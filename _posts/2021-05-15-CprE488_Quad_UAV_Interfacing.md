---
title: CprE 488 Quad UAV Interface
date: 2021-05-15 8:00:00 +/-1111
author: Jake Vaughn
categories: [Projects, Iowa State University]
tags: [project, ISU, portfolio]
---

![quadUAV](/images/488/quadUAV.png) | ![remote](/images/488/remote.png)


## Overview:

Our goal for this project was to design an interfacing and control platform for a simple quadcopter UAV and demonstrate its effectiveness. The project can be found on my [github](https://github.com/jake-vaughn/CPRE-488-MP1).

## Concepts

Concepts that we learned were
- Interfacing – Reverse engineering of an RC transmitter to gain an understanding of the trainer interface and format. 
- IP core design and integration – Use of Xilinx tools to generate an AMBA AXI4-compliant IP core that will integrate into an existing XPS project. 
- Finite State Machine design – Design and implement an FSM-based hardware module to record and transmit PPM data.


## Description

In this project we were tasked with intercepting, deciphering, and relaying PPM (Pulse Position Modulation) data signals from the quadcopter to the controller and vice-versa. To do this we need to use a Finite State Machine on an FPGA board using Verilog.  The software side was a program rc_control that was able to handle and switch between modes with physical switches. Modes such as hardware relay, software relay, software debug, software record, and software replay. To accomplish the task we used a heap load of documentation and prior knowledge.

My main focus in the group was understanding the PPM data signals and creating a finite state machine with verilog. The FSM was able read in a PPM signal and produce a PPM signal in hardware to relay to the quadcopter.

This was a very enjoyable project as I am very fond of working with Verilog.