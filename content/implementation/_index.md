---
title: "Implementation"
date: 2022-12-11T16:26:50-08:00
draft: true
math: true
---

# Hardware

{{< figure src="../img/fish-design.png" title="Hardware layout of FishBot" height=100% width=100% >}}

Our hardware implementation consists of several key components as highlighted in the above figure. This includes:

- **Underactuated Soft Tail** The tail mechanism is the primary actuator for our robot. It consists of a series of rigid segmented "ribs" joined together by revolute joints. From a kinematic perspective, this forms a multi-bar linkage system with a high degree of freedom. However, this entire mechanism is actuated by a single capstan cable drive connected to our servomotor. When tension is applied in either direction in the cable, an overall deflection left/right is respectively produced in the overall tail. This highly underactuated assembly is then encased by a soft flexible sleeve cast from silicone rubber (Smooth-On DragonSkin 10). This sleeve acts as the caudal fin of our fish robot, enabling undulatory propulstion through the water. 

{{< figure src="../img/logistic-map.png" title="Logistic mapping for tail angle" height=100% width=100% >}}

- **Waterproof Servomotor** A waterproof servomotor is sourced with appropriate specifications to meet both speed and torque requirements for driving the tail mechanism. After assessing potential solutions, we utilize a FlashHobby 35kg metal geared servo from Amazon, with specifications listed below.
INSERT SERVO TABLE/FIGURE
- **Electronics Compartment** A waterproof compartment is used to house a 2S 7.4V LiPo battery used to power our servomotor. This compartment is fabricated from standard PLA material using a desktop FDM 3D printer. The part is then treated for water resistance by painting with an XTC-3D epoxy coating. A lid for easy access is laser cut from 1/4" acrylic, and secured to the compartment with M3 bolts and an O-ring gasket.
- **ArUco Tag** The pose of the FishBot is extracted from an ArUco tag affixed to the front of the robot. This is created by hand-painting a thin wooden sheet using a permanent marker in order to create a waterproof component (standard printer paper quickly dissolves and loses integrity when submerged).
- **Microcontroller (WiFi Antenna)** Lastly, a floating anchor is created using a small section of pool noodle. The FishBot is affixed to this anchor at its center of gravity in order to ensure proper balance in the water. The ESP32 microcontroller is then affixed to this float and waterproofed using a plastic sheet. This is done to ensure the antenna on our microcontroller is above water, since we determined that WiFi has substantial issues transmitting through water.

# Software

The project structure is as shown:

```
eecs-106a-project
├── LICENSE
├── README.md
└── src
    ├── CMakeLists.txt
    └── fishbot
        ├── CMakeLists.txt
        ├── launch
        │   └── init.launch
        ├── msg
        │   └── FishError.msg
        ├── package.xml
        └── src
            ├── control.py
            ├── cv.py
            ├── http_client.ino
            ├── http_server.py
            ├── motion_planner.py
            └── wifi_credentials.h
```

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

This node allows converts the location of a given target point and the current location of the fish into an error that the fish can control.
This is sent to the `/controller` node to FishBot via the `/motion_plan` topic.
It is communicated via a custom `FishError` message:

```
float64 distance_error
float64 angular_error
```

In order to read both the current position of the fish _and_ a target node, we have to subscribe to two different topics.
This is done idiomatically using a custom class:

```python
class TagTracker:
    fish_tag:       Pose     = field(default_factory=Pose)
    target_tag:     Pose     = field(default_factory=Pose)
    distance_error: float    = float("inf")
    angular_error:  float    = float("inf")
```

In order to determine if the fish is ready to move to the next control point, we insert this condition to the callback method `TagTracker.target_tag_callback`:

```python {hl_lines=["2-3"]}
def target_tag_callback(self) -> None:
    if abs(self.distance_error) <= 0.1:
        self.target_tag = next(self.control_pts)
    self.joint_callback()
```

If the distance error to the current node is within a given tolerance, it will iterate to the next control node.
Examining the `TagTracker.joint_callback` method, we first find the \\(xy\\)-plane positions of FishBot and the target point.
From there, we can derive the Euclidean distance error.
We can similarly derive the angular error after offsetting by FishBot's current orientation, derived from its quaternion pose.
The lines of interest are highlighted:

```python {hl_lines=["2-3", 10, "12-13"]}
def joint_callback(self) -> None:
    delta_x = self.fish_tag.position.x - self.target_tag.position.x
    delta_y = self.fish_tag.position.y - self.target_tag.position.y
    fish_quaternion = [
        self.fish_tag.orientation.x,
        self.fish_tag.orientation.y,
        self.fish_tag.orientation.z,
        self.fish_tag.orientation.w
    ]
    _, _, yaw = tft.euler_from_quaternion(fish_quaternion)

    self.distance_error = math.sqrt(delta_x**2 + delta_y**2)
    self.angular_error = math.atan2(delta_y, delta_x) - yaw

    if self.angular_error < -math.pi:
        self.angular_error += 2 * math.pi
    elif self.angular_error > math.pi:
        # Want to bound the right turn angular error at pi
        # Want to bound the left turn angular at -pi
        self.angular_error -= 2 * math.pi
```

### `control.py`

Utilizing the logistic map from our [design](../design/_index.md)

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
The result is saturated to safe angular displacements allowed by FishBot's motor.
<!-- logistic map stuff -->

# Wireless Communication

{{< figure src="../img/wifi-diagram.png" title="Diagram of wireless communication protocol" height=100% width=100% >}}

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

The ESP32 connects to a locally hosted server that will send the motion commands to the FishBot. By connecting to the server through Wi-Fi, the FishBot can persistently see the commands being fed by the control node from the ROS running on a host system. 
After reading the commands from the server, it decides on one of three motions for the FishBot: move straight forward, turn left by some angle, or turn right by some angle.
```cpp
String http_GET_request(const char* server) {
  HTTPClient http;
    
  // Your IP address with path or Domain name with URL path 
  http.begin(server);
  
  int response_code = http.GET();
  
  String payload = "--"; 
  
  if (response_code > 0) {
    payload = http.getString();
  } 
  http.end();
  
  return payload;
}
```

The ESP32 controls the servo motor. This means that we need to import the `"ESP32Servo.h"` library so that the microcontroller can control the servo motor using the appropiate functions provided by this library. 
In conjuction, the ESP32 needs to initialize/set up several components necessary for achieving the desired motion of the FishBot, including the wireless connection, the trim of the servo motor, and the desired frequency.

```cpp
void setup() {
  // put your setup code here, to run once:

  Serial.begin(115200);
  delay(10);

  WiFi.begin(SSID, PWORD);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
  }

  ESP32PWM::allocateTimer(0);
  ESP32PWM::allocateTimer(1);
  ESP32PWM::allocateTimer(2);
  ESP32PWM::allocateTimer(3);
  tail_motor.setPeriodHertz(50);
  tail_motor.attach(servo_pin, 500, 2400);
  tail_motor.write(trim);
  delay(1000);
}
```

After initializing the ESP32 with the necessary setup code, we run a loop that constantly downloads and parses the server's messages. This allows the controller to dynamically change the behavior of the servo motor, thereby influencing the FishBot's motion on the fly.

```cpp
void move_tail(int a, int b, unsigned long moving_time) {
  unsigned long moveStartTime = millis(); // time when start moving
  unsigned long progress = 0;

  while (progress <= moving_time) {
    progress = millis() - moveStartTime;
    long angle = map(progress, 0, moving_time, a, b);
    tail_motor.write(angle);
  }
}

void loop() {
  // put your main code here, to run repeatedly:
  String key;
  if (WiFi.status() == WL_CONNECTED) {
    key = http_GET_request(URL);
  }

  int str_len = key.length()+1;
  char commands[str_len];
  key.toCharArray(commands, str_len);
  
  char *cmd = strtok(commands, delim);
  char *param = strtok(NULL, delim);
  
  int angle = atoi(param);
  if (strcmp(cmd, frwd) == 0) {
    move_tail(trim, trim + 30, 200);
    move_tail(trim + 30, trim - 30, 400);
    move_tail(trim - 30, trim, 200);
  } else if (strcmp(cmd, turn) == 0) {
    tail_motor.write(trim + angle);
    delay(500);
    move_tail(trim + angle, trim, 500);
  }
}
```

## Computer Vision (CV)

The goal of the computer vision node is to take an image from the camera, process the image using the OpenCV library in Python, and detect the obstacles that are present in the tank. To implement this node, we first downsampled the image by a factor of 2 due to the size of the image and to improve computational speed: 

Then, we perform image processing using the functions in the OpenCV library. To do so, we create a mask by defining lower and upper limits of BGR values to specifically threshold out the blue values in the image, which are the color of the two obstacles: 

Then, we create a kernel and use it to remove some noise from the blue mask: 

Next, we use this final processed blue mask to form the threshold image of blue colors: 

Finally, we upsample the image by a factor of 2 back to the size of the original image before we did any processing: 

Here are the original and processed/thresholded images: 

As you can see, the code performed the color thresholding pretty well but there are some limitations that requires some additional improvements in the future. One limitation was that since we also had some blue tape on the edges of the tank, the code also caught some of that 

