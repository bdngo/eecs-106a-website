---
title: "Design"
date: 2022-12-11T15:55:36-08:00
draft: true
---

{{< figure src="../img/hw-design.png" title="Hardware design of FishBot" height=100% width=100% >}}

The mechanical design of our fish robot is largely based on the prior platform developed by Wang (me, Stanley!) et al.
The underactuated tail consists of 5 linkage segments actuated by a single cable drive, with a soft silicone sleeve for underwater propulsion.
However, our iteration consists of several notable improvements over the previous design.
Specifically, we make a large leap toward designing a tetherless and semi-autonomous robotic system.
From a hardware perspective, several key design choices are made:

- **Actuator**: A waterproof servo is utilized to avoid complex waterproofing of moving components.
The primary tradeoff with this is (a) expensive parts (these servos are $50+ each) and (b) limited selection of servomotors available.
Both drive speed and torque are important considerations here since the tail mechanism requires an adequate driving force and must bend at a high rate.
- **Electronics**: The primary control and power electronics (LiPo battery, microcontroller, etc.) must all be housed inside the system underwater to avoid an external tether.
This necessitates the design of a waterproof enclosure.
The tradeoff here is primarily the additional weight added from these components, which have a substantial impact on the buoyancy and balance of our robot.
- **Communications**: Wireless communication is necessary in order to connect our robot with our computer ROS system.
This is implemented using an HTTP server with our microcontroller.
The microcontroller is attached to the flotation device of the robot to ensure the antenna is not submerged, since WiFi does not travel well through the air-water interface.

The tradeoff here lies in the additional layer of complexity, which introduces another potential point of failure in the system.

From a realistic engineering perspective, our design choices lead to several advantageous and disadvantageous outcomes:

- The waterproofing of the system is **highly robust** due to the waterproof servo and well-designed enclosure.
Minimal leakage is observed even after 

{{< figure src="../img/system-design.png" title="Overall system design of FishBot" height=100% width=100% >}}
