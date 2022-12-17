---
title: "Results"
date: 2022-12-11T15:55:44-08:00
draft: true
---

We can define three separate stages of development for FishBot.

# 1. ArUco Tag-Based Target Acquisition

The robotic fish is able to intelligently go from any starting position to the desired goal position we give it using an ArUco tag to pinpoint the robot's location and a camera for spatializing the environment.
In order for the fish to decide what direction it should be moving in and how much it should turn by, it calculates the Euclidean distance from its ArUco tag to the target ArUco tag and the angular difference between the tags.
Using the calculated values, the robotic fish can intelligently decide if it needs to turn left, right, or move forward. When turning in either direction, the gain used to determine how much to turn is the angular error itself which worked very well. This attribute can be seen in the video below.
In the video below, we see how this works in the real-world application of the FishBot where it persistenly follows the target ArUco tag dynamically changing directions when needed.

{{< youtube SNpgfCUs7Ck >}}

# 2. Target Node-Based Motion Planning

FishBot is able to reach a set of target nodes in rapid succession using its control systems.
In order for the FishBot to achieve its goal position we spatialized the enviornnment with the webcam we used and pre-defined control points throughout the tank for the fish to try and reach. After the FishBot reaches its current target position, the target position iterates to the next one until the FishBot has reached its goal position.
The dynamics of the FishBot was improved to incorporate the logistic curve fit of the servo motor being used for the angle of the tail of the FishBot with respect to the angle of the servo motor. Doing so led to more accurate turns when the FishBot flicks its tail to move in one direction or the other.
In the video below, we can visually see the calibrated control-points that the FishBot maneuvers to in order to reach the final target position.

{{< youtube 4HTXYV4v8Os >}}
