# Orphaned package warning
These packages are no longer maintained and will not work with newer versions of ROS and Gazebo.
The latest ROS distribution supported is ROS2 Humble.
Feel free to fork and continue development (BSD license).

# Velodyne Simulator
URDF description and Gazebo plugins to simulate Velodyne laser scanners

![rviz screenshot](img/rviz.png)

## Dependencies

This workspace has been verified with ROS 2 Humble and Gazebo Classic.

Install the ROS packages used by the launch files and plugin with:

```bash
sudo apt update
sudo apt install \
    ros-humble-gazebo-dev \
    ros-humble-gazebo-ros-pkgs \
    ros-humble-launch \
    ros-humble-launch-ros \
    ros-humble-robot-state-publisher \
    ros-humble-rviz2 \
    ros-humble-urdf \
    ros-humble-xacro
```

If you are setting up a fresh ROS environment, `rosdep` can install the same dependencies automatically once the ROS 2 apt sources are configured:

```bash
rosdep install --from-paths . --ignore-src -r -y
```

## Build

```bash
source /opt/ros/humble/setup.bash
cd ~/velodyne_simulator_ros2_humble
colcon build --symlink-install
source install/setup.bash
```

Re-source `install/setup.bash` in any new terminal before running the launch files.

## Run

Launch the example robot, Gazebo, and RViz:

```bash
ros2 launch velodyne_description example.launch.py
```

Useful launch arguments:

```bash
ros2 launch velodyne_description example.launch.py gpu:=true
ros2 launch velodyne_description example.launch.py gui:=false
ros2 launch velodyne_description example.launch.py organize_cloud:=true
```

# Features
* URDF with colored meshes
* Gazebo plugin based on [gazebo_plugins/gazebo_ros_ray_sensor](https://github.com/ros-simulation/gazebo_ros_pkgs/blob/foxy/gazebo_plugins/src/gazebo_ros_ray_sensor.cpp)
* Publishes PointCloud2 with same structure (x, y, z, intensity, ring, time)
* Simulated Gaussian noise
* GPU acceleration (with known issues)
* Supported models:
    * [VLP-16](velodyne_description/urdf/VLP-16.urdf.xacro)
    * [HDL-32E](velodyne_description/urdf/HDL-32E.urdf.xacro)
    * Pull requests for other models are welcome
* Experimental support for clipping low-intensity returns

# Parameters
* ```*origin``` URDF transform from parent link.
* ```parent``` URDF parent link name. Default ```base_link```
* ```name``` URDF model name. Also used as tf frame_id for PointCloud2 output. Default ```velodyne```
* ```topic``` PointCloud2 output topic name. Default ```/velodyne_points```
* ```hz``` Update rate in hz. Default ```10```
* ```lasers``` Number of vertical spinning lasers. Default ```VLP-16: 16, HDL-32E: 32```
* ```samples``` Nuber of horizontal rotating samples. Default ```VLP-16: 1875, HDL-32E: 2187```
* ```organize_cloud``` Organize PointCloud2 into 2D array with NaN placeholders, otherwise 1D array and leave out invalid points. Default ```false```
* ```min_range``` Minimum range value in meters. Default ```0.9```
* ```max_range``` Maximum range value in meters. Default ```130.0```
* ```noise``` Gausian noise value in meters. Default ```0.008```
* ```min_angle``` Minimum horizontal angle in radians. Default ```-3.14```
* ```max_angle``` Maximum horizontal angle in radians. Default ```3.14```
* ```gpu``` Use gpu_ray sensor instead of the standard ray sensor. Default ```false```
* ```min_intensity``` The minimum intensity beneath which returns will be clipped.  Can be used to remove low-intensity objects.

# Known Issues
* At full sample resolution, Gazebo can take up to 30 seconds to load the VLP-16 plugin, 60 seconds for the HDL-32E
* When accelerated with the GPU option, ranges are heavily quantized ([image](img/gpu.png))
    * Solution: Use CPU instead of GPU
* Gazebo cannot maintain 10Hz with large pointclouds
    * Solution: User can reduce number of points (samples) or frequency (hz) in the urdf parameters, see [example.urdf.xacro](velodyne_description/urdf/example.urdf.xacro)
* Gazebo crashes when updating HDL-32E sensors with default number of points. "Took over 1.0 seconds to update a sensor."
    * Solution: User can reduce number of points in urdf (same as above)

