---
title: "Implementation"
date: 2022-12-11T16:26:50-08:00
draft: true
math: true
---

# Hardware

{{< figure src="../img/fish-design.png" title="Hardware layout of FishBot" height=100% width=100% >}}

# Software

## ROS

The current ROS workflow can be entirely run with the following command (given `roscore` is running and the proper files have been sourced):
```bash
roslaunch fishbot init.launch
```
Analyzing the launch file,
```xml {hl_lines=["8-9", 33, 45]}
<launch>
    <arg name="video_device" default="/dev/video0" />
    <arg name="image_width" default="640" />
    <arg name="image_height" default="480" />

    <arg name="markerId1" default="582"/>
    <arg name="markerId2" default="100"/>
    <arg name="markerSize1" default="0.06"/> <!-- in meter -->
    <arg name="markerSize2" default="0.08"/> <!-- in meter -->
    <arg name="eye" default="left"/>
    <arg name="marker_frame1" default="fish_frame"/>
    <arg name="marker_frame2" default="target_frame"/>
    <arg name="ref_frame" default=""/> <!-- leave empty and the pose will be published wrt param parent_name -->
    <arg name="corner_refinement" default="LINES" /> <!-- NONE, HARRIS, LINES, SUBPIX -->

	<node name="usb_cam" pkg="usb_cam" type="usb_cam_node" output="screen" >
		<param name="video_device" value="$(arg video_device)" />
		<param name="image_width" value="$(arg image_width)" />
		<param name="image_height" value="$(arg image_height)"/>
		<param name="pixel_format" value="mjpeg" />
		<param name="camera_frame_id" value="usb_cam" />
		<param name="io_method" value="mmap"/>
	</node>

	<node pkg="aruco_ros" type="single" name="aruco_fish">
        <remap from="/camera_info" to="/usb_cam/camera_info" />
        <remap from="/image" to="/usb_cam/image_raw" />
        <param name="image_is_rectified" value="True"/>
        <param name="marker_size" value="$(arg markerSize1)"/>
        <param name="marker_id" value="$(arg markerId1)"/>
        <param name="reference_frame" value="$(arg ref_frame)"/> <!-- frame in which the marker pose will be refered -->
        <param name="camera_frame" value="base_link1"/>
        <param name="marker_frame" value="$(arg marker_frame1)" />
        <param name="corner_refinement" value="$(arg corner_refinement)" />
    </node>

    <node pkg="aruco_ros" type="single" name="aruco_target">
        <remap from="/camera_info" to="/usb_cam/camera_info" />
        <remap from="/image" to="/usb_cam/image_raw" />
        <param name="image_is_rectified" value="True"/>
        <param name="marker_size" value="$(arg markerSize2)"/>
        <param name="marker_id" value="$(arg markerId2)"/>
        <param name="reference_frame" value="$(arg ref_frame)"/> <!-- frame in which the marker pose will be refered -->
        <param name="camera_frame" value="base_link2"/>
        <param name="marker_frame" value="$(arg marker_frame2)" />
        <param name="corner_refinement" value="$(arg corner_refinement)" />
    </node>
    
    <node name="motion_planner" pkg="fishbot" type="motion_planner.py" output="screen" />
    <node name="controller" pkg="fishbot" type="control.py" output="screen" />
</launch>
```
The launch file first initializes several variables.
Notable ones highlighted are `markerSize1` and `markerSize2`, which tell `aruco_ros` how large the tags are assumed to be on a side.
Then, both the `usb_cam` and `aruco_ros` nodes are launched.
Two `aruco_ros` nodes are launch to represent the fish's tag, as well as the tag of a target to follow.
The topics they publish two are highlighted.
This initializes the raw camera feed and enables ArUco tag detection.
Finally, the custom `motion_planner` and `controller` nodes are launched, creating the full ROS workflow.

### `cv.py`

### `motion_planner.py`

### `control.py`

Given the angular error \\(e_\theta\\), the equation used is
\\[
    |S| =
    \begin{cases}
        S_{max} & |e_\theta| > S_{max} \\\\
        S_{min} & |e_\theta| < S_{min} \\\\
        e_\theta \cdot \frac{180}{\pi}
    \end{cases}
\\]
where the sign is determined by the direction of desired motion.
<!-- logistic map stuff -->

# Wireless Communication

The microcontroller on the ESP32 _does not_ run ROS.
The exact commands it needs to run are calculated on a host computer and transmitted wirelessly.
This is done via a WiFi interface.
The result of the `/controller` node is translated into an angular position to drive FishBot's tail to.
The resultant "strength" is written to a file on disk with the following grammar:
```bnf
<command> ::= "DONE\n0" | "FWRD\n0" | <turn>
<turn> ::= "TURN\n" n
```
where \\(n\\) is some integer between \\(\pm 50\\).
For example, if we wanted to tell FishBot to move the tail 34° to the right, then assuming rightwards is positive, the file would look like:
```
TURN
34
```
While the ROS nodes are running, there is a Python HTTP server also running.
This server hosts the exact file to the LAN, which any computer connected to the same network can also access.
Every clock cycle of the ESP32, it queries this URL.
The payload is then parsed to determine the exact command FishBot can execute as well as the strength of it, if it is a turn.
This setup has a latency on the order of hundreds of milliseconds.


## ESP32
