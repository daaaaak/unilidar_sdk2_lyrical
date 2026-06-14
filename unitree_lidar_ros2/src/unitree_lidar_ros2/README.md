This is a ROS2 package for Unitree Lidar L2.

## ROS2 Lyrical on Ubuntu 26

This package has compatibility updates for ROS2 Lyrical on Ubuntu 26.

Install the required packages:

```bash
sudo apt update
sudo apt install libpcl-dev ros-lyrical-pcl-conversions ros-lyrical-tf2-ros
```

Build from the ROS2 workspace root for this package:

```bash
cd /home/pi/unilidar_sdk2_lyrical/unitree_lidar_ros2
rm -rf build install log
source /opt/ros/lyrical/setup.bash
colcon build --symlink-install
```

The Lyrical package layout may install headers under nested include paths, for example:

```text
/opt/ros/lyrical/include/pcl_conversions/pcl_conversions/pcl_conversions.h
/opt/ros/lyrical/include/tf2_ros/tf2_ros/transform_broadcaster.h
```

The source uses those nested include paths for `pcl_conversions` and `tf2_ros`. The old unused `tf2/LinearMath/Quaternion.h` include was removed because the node publishes quaternion values directly from the SDK IMU packet.

The CMake file also avoids `ament_target_dependencies()` for Lyrical environments where that macro is unavailable even after sourcing `/opt/ros/lyrical/setup.bash`. It links ROS2 package include directories and libraries through local target-based CMake logic instead.
