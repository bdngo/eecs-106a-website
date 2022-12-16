---
title: "Conclusion"
date: 2022-12-11T15:55:50-08:00
math: true
draft: true
---

# Challenges

- Inherent system dynamics
    - Water + Underactuated System!
    - Nonlinear/Nonholonomic \\(\to\\) Highly unpredictable and difficult to model!
    - Accounted for by lowering demand for control precision

Communicating with the ESP32 while it was inside the fish underwater was a major challenge for us as the WiFi signals were having trouble being received by the ESP32.
As a result, we went through a couple iterations of different solutions on tackling the problem, ultimately leading us to keeping the ESP32 floating on top of the water.
While this solved our issue, water did leak into the ESP32’s carrier multiple times resulting in it being fried.
Another challenge was using a "waterproof" servo motor that can only last for a finite amount of time under water.
Throughout the entirety of this project, we went through a minimum of three servo motors as the water eventually causes damage to them.


# Future Work

The next stage of our research project is to implement the CV with an occupancy grid to improve the motion planner.
Doing so allows us to remove the control points we calibrated ahead of time ensuring that the robotic fish can dynamically find its path to the goal position.
For determining the optimal trajectory, we can use algorithms such as A* and RRT to solve for this.
Next, we need to test the robotic fish in several environment configurations after the implementation of an improved motion planner.
This includes adding or removing the amount of obstacles in the tank, placing them in different positions, and altering the size of the obstacles.
Finally, having a much larger tank in terms of width and length would significantly improve our research as we are currently confined to work in the smaller tank that we have.
A larger tank will allow us to test various path-planning and motions of the robotic fish such that it won’t constantly be hitting the walls of the tank.
