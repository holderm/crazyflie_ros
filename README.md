crazyflie_ros
=============

ROS Driver for Bitcraze Crazyflie (http://www.bitcraze.se/), with the following features:

* Uses the official python SDK (which needs to be installed/copied separately)
* Publishes on-board sensors in ROS standard message formats
* Supports ROS parameters to reconfigure crazyflie parameters

Ashfaq fixed the CMake Install files and I give a couple of hints how to get the Crazyflie working with ROS.
You find the original repo at
https://github.com/whoenig/crazyflie_ros

## Installation

Clone the package into your catkin workspace:
```
git clone https://github.com/smarties181/crazyflie_ros.git
```

Additionally, you should have the bitcraze SDK on your file system.
See https://github.com/bitcraze/crazyflie-clients-python for details.
Adapt the setting by open teleop_xbox360.launch
  <arg name="crazyflieSDK" default="~/crazyflie/crazyflie-clients-python/lib" />
  <arg name="uri" default="radio://0/60/250K" />  <- adapt this
  <arg name="joy_dev" default="/dev/input/js1" /> <- adapt this

If you want to use joystick teleoperation, you should setup the hector_quadrotor package (http://wiki.ros.org/hector_quadrotor).

Now  install the package
```
catkin_make_isolated --install --force-cmake
source /home/"yourusername"/catkin_ws/install_isolated/-setup.bash
```
Make sure you have the Crazyflie SDK installed. You'll find it here:
```
https://github.com/bitcraze/crazyflie-clients-python
```
Here you have to make some changes in order to install the cfclient libary correctly:
open setup.py file, on line 63 you find cfclient.utils.inputdevices rename it to cfclient.utils.inputreaders

make setup.sh executeable and install it:
```
sudo ./setup.sh
```

## Usage

There are two packages included: crazyflie and crazyflie_demo.
Crazyflie contains the driver and a launch file which can be used in your own projects.

Crazyflie_demo contains a small demo for teleoperating the crazyflie with a joystick and visualizing the sensor data in rviz.

You can use
```
roslaunch crazyflie_demo teleop_xbox360.launch
```
to teleoperate the crazyflie.
Please note that there are (optional) arguments to change the device uri and path to the sdk.
See `crazyflie_demo/launch/teleop_xbox360.launch` for details.

## ROS Features

### Parameters

The launch file supports the following arguments:
* crazyflieSDK: Path to the official SDK from Bitcraze, e.g. ~/crazyflie/crazyflie-clients-python/lib
* uri: Specifier for the crazyflie, e.g. radio://0/80/2M
* tf_prefix: tf prefix for the crazyflie frame(s)
* roll_trim: Trim in degrees, e.g. negative if flie drifts to the left
* pitch_trim: Trim in degrees, e.g. negative if flie drifts forward

See http://wiki.bitcraze.se/projects:crazyflie:userguide:tips_and_tricks for details on how to obtain good trim values.

### Subscribers

#### cmd_vel

Similar to the hector_quadrotor, package the fields are used as following:
* linear.y: roll [e.g. -30 to 30 degrees]
* linear.x: pitch [e.g. -30 to 30 degrees]
* angular.z: yawrate [e.g. -200 to 200 degrees/second]
* linear.z: thrust [10000 to 60000 (mapped to PWM output)]

### Publishers

#### imu
* sensor_msgs/IMU
* contains the sensor readings of gyroscope and accelerometer
* The covariance matrices are set to unknown
* orientation is not set (this could be done by the magnetometer readings in the future.)
* update: 10ms (time between crazyflie and ROS not synchronized!)
* can be viewed in rviz

#### temperature
* sensor_msgs/Temperature
* From Barometer (10DOF version only) in degree Celcius (Sensor readings might be higher than expected because the PCB warms up; see http://www.bitcraze.se/2014/02/logging-and-parameter-frameworks-turtorial/)
* update: 100ms (time between crazyflie and ROS not synchronized!)

#### magnetic_field
* sensor_msgs/MagneticField
* update: 100ms (time between crazyflie and ROS not synchronized!)

#### pressure
* Float32
* hPa (or mbar)
* update: 100ms (time between crazyflie and ROS not synchronized!)

#### battery
* Float32
* Volts
* update: 100ms (time between crazyflie and ROS not synchronized!)

## Similar Projects

* https://github.com/gtagency/crazyflie-ros
  * no documentation
  * no teleop
* https://github.com/omwdunkley/crazyflieROS
  * coupled with GUI
  * based on custom firmware
* https://github.com/mbeards/crazyflie-ros
  * incomplete
* https://github.com/utexas-air-fri/crazyflie_fly
  * not based on official SDK
  * no support for logging
* https://github.com/mchenryc/crazyflie
  * no documentation

## Notes

* The dynamic_reconfigure package (http://wiki.ros.org/dynamic_reconfigure/) seems like a good fit to map the parameters, however it has severe limitations:
  * Changed-Callback does not include which parameter(s) were changed. There is only a notion of a level which is a simple bitmask. This would cause that on any parameter change we would need to update all parameters on the Crazyflie.
  * Parameters are statically generated. There are hacks to add parameters at runtime, however those might not work with future versions of dynamic_reconfigure.
  * Groups not fully supported (https://github.com/ros-visualization/rqt_common_plugins/issues/162; This seems to be closed now, however the Indigo binary packages did not pick up the fixes yet).
