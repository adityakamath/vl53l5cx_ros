# vl53l5cx_ros
ROS 2 node for the [VL53L5CX](https://www.sparkfun.com/products/18642) ToF (8x8 Array) Imager. 

Note: This implementation is a bit over-engineered, as I have been experimenting with ROS 2 [managed/lifecycle](https://design.ros2.org/articles/node_lifecycle.html) nodes, [executors](https://docs.ros.org/en/humble/Concepts/About-Executors.html#executors) and [composition](https://github.com/ros2/examples/blob/rolling/rclpy/executors/examples_rclpy_executors/composed.py) using Python.

## Implementation details

* ```tof_imager_publisher```: This executable uses the [vl53l5cx_python](https://github.com/Abstract-Horizon/vl53l5cx_python/tree/main) library to access sensor data over I2C. Distance measurements from the sensor are converted to [Pointcloud2](https://docs.ros2.org/foxy/api/sensor_msgs/msg/PointCloud.html) messages which are published periodically using a timer to the ```pointcloud``` topic. The sensor has a resolution of 8x8 as default. While the sensor can also be used with a resolution of 4x4, this package currently does not support it. This implementation is designed as a lifecycle component and can be run individually as well.
* ```tof_imager_node```: This executable creates an instance of ```tof_imager_publisher``` and runs it using a single threaded executor. 

* ```tof_imager_launch.py```: This is the launch file that launches ```tof_imager_node``` as a  lifecycle node, loads its parameters, and then configures and activates it. The lifecycle node is first initialized, then set to 'configure' from the launch file. When the 'inactive' state is reached, the registered event handler activates the node.

## Parameters

* ```pointcloud_topic```: Pointcloud2 message topic name (Default: pointcloud)
* ```frame_id```: Parent frame for the Pointcloud2 message (Default: tof_frame)
* ```mode```: 1 for Continuous mode OR 3 for Autonomous mode (Default: 1)
* ```ranging_freq```: Ranging frequency, limited to 15Hz (Default: 15)
* ```timer_period```: Timer period in seconds (Default: 0.1)

More info about the mode and ranging frequency parameters can be found in the [Datasheet](https://www.st.com/resource/en/datasheet/vl53l5cx.pdf) and [Guide](https://www.st.com/resource/en/user_manual/um2884-a-guide-to-using-the-vl53l5cx-multizone-timeofflight-ranging-sensor-with-wide-field-of-view-ultra-lite-driver-uld-stmicroelectronics.pdf)

## How to use

* Connect the VL53L5CX to the I2C GPIO pins of a Raspberry Pi device:
  * 3.3V to any 3.3V pin
  * SDA to BCM 2
  * SCL to BCM 3
  * GND to any ground pin
* Install the vl53l5cx library:
  ```
  $ git clone https://github.com/Abstract-Horizon/vl53l5cx_python
  $ cd vl53l5cx_python
  $ sudo python3 setup.py install
  ```
* Clone this repository in a ROS 2 workspace. Check the ```sensor_params.yaml``` file in the config directory, and make any necessary changes.
* Build the package and run the launch file: ```ros2 launch vl53l5cx_ros tof_imager_launch.py```

## Results

This package was tested using a [VL53L5CX ToF Imager](https://www.sparkfun.com/products/18642) from Sparkfun and ROS 2 Humble on two devices:
* A Raspberry Pi 4 (8GB) running Ubuntu 22.04 with a real-time kernel
* A Raspberry Pi Zero 2 W running Ubuntu 22.04 without any kernel modifications
In both cases, and for both modes a max output frequency of ~6.1Hz was achieved. 





