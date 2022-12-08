---
title: Custom ASIC Temperature Sensor
date: 2020-12-14 8:00:00 +/-1111
author: Jake Vaughn
categories: [Projects, Iowa State University]
tags: [project, ISU, portfolio, IEEE, ASIC, ModelSim, Innouvus, Verilog]
---

# Overview:
In this final project for CprE 465, I set out to design and implement a custom ASIC (Application-Specific Integrated Circuit) that could be used as a temperature sensor. The resulting ASIC would be a specialized piece of hardware optimized for temperature sensing, and could be easily integrated into a larger system. The design of the ASIC was optimized for power, area, and timing by implementing it in low-level logic at the micrometer scale. This was necessary in order to achieve the performance and efficiency required for the task, as opposed to implementing it in software on a processor.

## Description
The device will calculate up to 12 temperature readings at a time, prioritizing the most recent values entered by the user. Once the user stops inputting values, the device will compute the moving average or standard deviation for the last 12 values, depending on the operating mode. Once the calculation is complete, the device will send a signal to the user indicating that it is ready for a new data set. The device also makes use of a guess value to make future calculations faster.

## Process
To build the ASIC, I used verilog, a hardware description language that allows designers to specify the functionality of digital circuits at a high level of abstraction. I used modelsim to simulate the design, which allowed me to verify that the ASIC behaved as expected before moving on to the implementation phase.

![modelSim-example](/images/465/modelSim.png)

One major challenge in the project was the need for division calculations. The design of an ASIC that can perform division and be synthesized for real hardware is not a simple task. The required size for a fully functioning divider would be too large for the project requirements, and I wanted the project to be feasible for manufacturing. My solution was to approximate division by converting to IEEE floating point numbers and using multiplication instead. Through much maneuvering, I was able to achieve very accurate results from the division calculations using this approach.

Once I was satisfied with the simulation results, I used innouvus to implement the design on a physical ASIC. Innouvus helped to translate the high-level verilog description of the ASIC into a lower-level representation that could be used to physically manufacture the circuit. Innouvus is a tool that is commonly used in the design of ASICs and other specialized digital circuits.

During the implementation phase, innouvus automatically synthesized the verilog design into a gate-level representation, which is a representation of the circuit at the level of individual logic gates. This allowed me to verify that the design met the required performance constraints and would function correctly when implemented on the physical ASIC.

![Innouvus-diagram](/images/465/innovus-diagram1_orig.png)

Once the design was successfully synthesized, innouvus was used to generate the necessary files for manufacturing the physical ASIC. These files contained detailed instructions for fabricating the circuit on the target chip, and included information about the layout and routing of the individual components.

Overall, the use of innouvus in this project allowed me to efficiently and accurately implement the design of the temperature sensor ASIC, ensuring that it would function correctly when manufactured and used in a real system.

## Conclusion

In this project, I designed an ASIC temperature sensor that is capable of calculating the moving average or standard deviation of a given set of temperatures. I utilized advanced design tools and methodologies to create a specialized piece of hardware optimized for temperature sensing. The resulting ASIC is able to measure temperature in real-time, making it a valuable addition to any system that requires fast and reliable temperature sensing.

Through the course of this project, I gained valuable experience with object-oriented design and Verilog techniques, including pipelining and power optimization. I used these skills to design a temperature sensor device that met all specifications and passed Genus and innouvus simulations. Although the chip was not physically manufactured due to COVID-19 restrictions, I made sure that the design was suitable for synthesization, so it could potentially be manufactured in the future.