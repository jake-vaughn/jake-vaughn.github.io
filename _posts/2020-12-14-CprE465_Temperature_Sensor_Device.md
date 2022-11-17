---
title: CprE 465 Temperature Sensor Device
date: 2020-12-14 8:00:00 +/-1111
author: Jake Vaughn
categories: [Projects, Iowa State University]
tags: [project, ISU, portfolio, IEEE, ASIC, ModelSim, Innouvus, Verilog]
---

# Overview:
In this project, I designed an ASIC that can take any number of random input temperatures and calculate the Moving average or standard deviation. In order to optimize for power, area, and timing the design of the device needed to be an ASIC rather than a software program run on a processor. To accomplish this I need to solve the problem of creating moving averages and standard deviations in as low-level logic as possible. The end design of the ASIC was in the micrometer scale.

![Innovus-diagram](/images/465/innovus-diagram1_orig.png)

## Description
The device will calculate up to 12 temperature readings at a time, prioritizing the last 12 temperatures entered by the user. Once the user stops inputting values depending on the operating mode either the moving average or standard deviation will be computed for the 12 values. After calculation, a signal flag will be sent to the user saying it has completed the solution through the wire vector AVG/SD. Once it has been done and is ready for use again it will send a SAMPLE flag to the user for a clock cycle. Once the user gets the sample flag they can input fresh new data set to calculate. Each future reading of standard deviation then uses a guess value which makes it faster to find future readings.

![modelSim-example](/images/465/modelSim.png)

One major hurdle of the project was calculations that required division. This is because designing an ASIC that can do division and still be synthesizable on real hardware is no easy feat. The required size for making a fully working divider would be too big for my project and I wanted my project to be able to be made into a real die. My solution was to approximate division by converting to IEEE floating point numbers and using multiplication. With a lot of learning, I was able to get very accurate output from division calculations.

## Conclusion
I learned a significant amount about object-oriented design using modular components to get an end product. I learned how to use Verilog techniques to pipeline and minimize area and power consumption. The device worked to retrieve data from the user and calculate Moving Average or Standard Deviation as requested. Genus and innouvus simulations ran flawlessly, confirming that the temperature sensor device was operational. Sadly our class did not get the option to get our projects turned into real physical chips due to covid restrictions. Even though this was the case I made sure that my design passed synthesization test for a real-world chip.