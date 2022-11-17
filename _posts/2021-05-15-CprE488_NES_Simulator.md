---
title: CprE 488 NES Simulator on a Xilinx Zedboard FPGA board
date: 2021-05-15 8:00:00 +/-1111
author: Jake Vaughn
categories: [Projects, Iowa State University]
tags: [project, ISU, portfolio]
---

![NES Zelda](/images/488/nes-zelda.jpg)

## Overview:

During CprE 488 we were tasked with creating an NES Simulator with an FPGA board’s processor and input-output connections from said board. The project can be found on my [github](https://github.com/jake-vaughn/CPRE-488-MP0).

## Tools

Tools that we used were

- VIVADO: development environment for the hardware portion.
- Xilinx Software Development Kit (SDK): software development tool called from vivado used for c/c++ embedded software applications.
- Software and Hardware IP: various embedded / soft IP for Xilinx embedded processors and peripherals; compilers, drivers, and libraries for embedded software development.
- Putty: For serial port connection from windows

## Description

Our project mostly consisted of connecting different peripherals to the processor to run the nes bootloader. The bootloader itself of was stored on an sd card. Output from the FPGA was sent to 640x480 VGA display. The games were controlled by two simulated virtual controllers.

The most difficult part of this project was in getting correct display output. For this we needed to use

- Video Direct Memory Access (VDMA) peripheral – axi_vdma – which provides high-bandwidth direct memory access between a memory component (typically DRAM) and the AXI Stream video protocol.
- Video Timing Controller (VTC) peripheral – v_tc – generates the necessary timing signals for video out, including horizontal and vertical synchronization pulses and blanking timing.
- Video Out peripheral – axi4s_vid_out – interfaces with a video source using the AXI Stream protocol and in conjunction with a VTC core, produces the appropriate video output.
