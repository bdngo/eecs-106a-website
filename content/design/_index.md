---
title: "Design"
date: 2022-12-11T15:55:36-08:00
draft: false
---

# Hardware Design

A large part of our project lies in the mechanical design of our underwater system. Several key categories for functional requirements include:

- **Waterproof**: The design must be capable of prolonged submersion in water without operational failure.
- **Locomotion**: the design must be capable of dexterous swimming, consisting of both straight and turning maneuvers.
- **Reliability**: the system must be relatively simple to fix, maintain, and debug. The design should be highly modular and adaptable for future changes.

{{< figure src="../img/tail-design.png" title="FishBot hardware design features" height=100% width=100% >}}

The hardware for our fish robot is largely based on a prior platform developed by Wang (me, Stanley!) et al. 
We explore a robot with an underactuated tail consisting of 5 linkage segments actuated by a single cable drive, with a soft silicone sleeve for underwater propulsion. 
However, our iteration consists of several notable improvements over the previous design.
Specifically, we make a large leap toward designing a tetherless and semi-autonomous robotic system.
From a hardware perspective, several key design choices are made:

- **Actuator**: A waterproof servo is utilized to avoid complex waterproofing of moving components.
The primary tradeoff with this is
    1. expensive parts (these servos are $50+ each)
    2. limited selection of servomotors available

Both drive speed and torque are important considerations here since the tail mechanism requires an adequate driving force and must bend at a high rate.
- **Electronics**: The primary control and power electronics (LiPo battery, microcontroller, etc.) must all be housed inside the system underwater to avoid an external tether.
This necessitates the design of a waterproof enclosure.
The tradeoff here is primarily the additional weight added from these components, which have a substantial impact on the buoyancy and balance of our robot.
- **Communications**: Wireless communication is necessary in order to connect our robot with our computer ROS system.
This is implemented using an HTTP server with our microcontroller.
The microcontroller is attached to the flotation device of the robot to ensure the antenna is not submerged, since WiFi does not travel well through the air-water interface.
The tradeoff here lies in the additional layer of complexity, which introduces another potential point of failure in the system.

From a realistic engineering perspective, our design choices lead to several advantageous and disadvantageous outcomes:

- The waterproofing of the system is highly **robust** due to the use of a waterproof servo and well-designed enclosure. Minimal leakage is observed even after extended submersion in water for more than 60 minutes.
- The **durability and safety** of the system is extremely high.
There are no dangerous parts due to the intrinsic nature of the soft robot. Our robot has also survived severe drops onto the concrete lab floor without any damage.
- The **efficiency** of our system is not optimal for realistic marine deployment. The extremely small battery used for our microcontroller must be recharged frequently for prolonged use.

{{< figure src="../img/system-design.png" title="Overall ROS + Arduino system design of FishBot" height=100% width=100% >}}

{{< figure src="../img/motion-planning.png" title="Theoretical path planning of FishBot" height=100% width=100% >}}

# Software Design

Much of our project relies on a fully-functioning software interface that can perform the majority of the control and planning computations, and communicate with the FishBot.
In particular, we must be able to interface with a USB camera, perform planning computations based on the data, and reliably send and recieve wireless control messages over a local network. 

- **Modularity**: The structure of the software needs to be modular such that segments of the code base can be reused/repurposed for other use.
To do so, we will need to separate the various responsibilities of the software into different files and functions that can serve multiple purposes. 
This also helps provide abstractions between the different components of the software, making testing and debugging much easier.
Generally, this is good software practice so that future iterations of the design can reuse the prior code implementions.

- **Goals**: We want to handle the control, motion planning, and use of the servo motor through the software.
In order to do so, we will be utilizing ROS to separate these responsibilities into individual nodes and ensuring that we are writing modular code.

- **Design Decisions and Tradeoffs**: An implication that we faced during the software design was creating a reliable connection between ROS and the ESP32. 
As the FishBot is working in a underwater enviornment, we decided to implement a web server that can take the outputs from ROS and provide them to the ESP32. 
Using a wireless communication method allows us to remove an external force on the FishBot and allows for more freedom of movement for the FishBot.
The tradeoff here is that the python server is not completely stable where the server has to be reloaded during test cycles since it would get stuck and crash.
Another tradeoff we made was running ROS on a host computer rather than on the FishBot itself. This was due to us not being able to obtain a Rasberry Pi as a result of the microcontroller shortage.
While running ROS on a host computer allowed us to move forward with using ROS to control the FishBot, it required us to have to feed wireless signals into an underwater robot using an ESP32.
This is not ideal as communicating with underwater robots has always been a challenge and this adds latency between the outputs of ROS and the microcontroller using that output to control the servo motor. 
We couldn't run ROS using an ESP32 as the ESP32 is not compatible with ROS1, and ROS2 has a learning curve that we did not have the time to overcome.