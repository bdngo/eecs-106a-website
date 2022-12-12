---
title: "Conclusion"
date: 2022-12-11T15:55:50-08:00
draft: true
---

# Challenges

- Inherent system dynamics
    - Water + Underactuated System!
    - Nonlinear/Nonholonomic → Highly unpredictable and difficult to model!
    - Accounted for by lowering demand for control precision
- Water is a highly challenging environment!
    - Leakages + Hardware Struggles
    - Underwater Wireless Communication (open research problem)
    - Fried 5 servo motors and 3 microcontrollers over the course of the project!!!

# Future Work

- Implement CV + Occupancy Grid to improve the motion planner
    - Will remove the need to find control points ahead of time
    - Perform improved object-avoidance planning using A*, RRT, etc. for determining the optimal trajectory
- Test the robotic fish in several environment configurations after the implementation of an improved motion planner
    - Bigger tank!

