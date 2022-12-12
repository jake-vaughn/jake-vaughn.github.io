---
title: FPGA-based NES Emulator
date: 2021-05-15 8:00:00 +/-1111
author: Jake Vaughn
categories: [Projects, Iowa State University]
tags: [project, ISU, portfolio]
---

![NES Zelda](/images/488/nes-zelda.jpg)

# Overview:
In this project for CprE488, my team and I were tasked with designing a new video game console that would outperform the Nintendo Entertainment System (NES) using VIVADO, a development environment for hardware design using Xilinx FPGA devices. Our team had a new FPGA-based platform that offered more complex designs than the NES, as well as superior embedded system design skills. Our goal was to prototype the hardware functionality needed to port an existing NES emulator as proof of concept for our new video game platform.

To do this, we used VIVADO to design the hardware portion of the embedded processor system, and Vitis, a software development tool that could be called from VIVADO, to develop the C/C++ code for the emulator. We also utilized the Software and Hardware IP provided by Xilinx, which included compilers, drivers, libraries, and documentation, to assist with the development process. After designing and implementing the hardware and software, we tested and verified the performance of the prototype emulator on the Xilinx Zedboard development board.

## Tools
- **VIVADO**: development environment for hardware design using Xilinx FPGA devices
- **Vitis**: software development tool for C/C++ embedded software development and verification
- **Software and Hardware IP**: compilers, drivers, libraries, and documentation for Xilinx embedded systems
- **Xilinx Zedboard development board**: for testing and verifying the performance of the prototype emulator

![Zynq Block Design](/images/488/Zynq-block-design.png)

## Description

To do this, we first set up our development environment by creating a new directory for the class and lab, and unzipping the provided MP0 starting directory structure. We then examined the C code implementation of the existing NES bootloader and emulator, starting with nes_bootloader.c.

Next, we used VIVADO to create a new hardware project and added the necessary peripherals to implement the 640x480 video output over VGA. Peripherals included were

- Video Direct Memory Access (VDMA) peripheral – axi_vdma – which provides high-bandwidth direct memory access between a memory component (typically DRAM) and the AXI Stream video protocol.
- Video Timing Controller (VTC) peripheral – v_tc – generates the necessary timing signals for video out, including horizontal and vertical synchronization pulses and blanking timing.
- Video Out peripheral – axi4s_vid_out – interfaces with a video source using the AXI Stream protocol and in conjunction with a VTC core, produces the appropriate video output.

We configured these peripherals with the appropriate parameters, such as the data output format and the framebuffer resolution.

![FPGA Connection Diagram](/images/488/Fpga-connection-diagram.png)

After successfully implementing the hardware system, we moved on to the software development using Vitis. We imported the existing NES emulator code into the Vitis workspace and set up the hardware platform in the software project. We then compiled and ran the emulator on the hardware platform, verifying that it was able to output the video display to a monitor.

## Conclusion
In conclusion, we successfully designed and implemented a hardware and software system for NES emulation on the Xilinx Zedboard development board using VIVADO and Vitis. We were able to generate a 640x480 video output over VGA and run the existing NES emulator code on the hardware platform.

The project can be found on my [github](https://github.com/jake-vaughn/CPRE-488-MP0).