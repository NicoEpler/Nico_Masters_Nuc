
# Installation and basics

1. Changes made for [installation](https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example?tab=readme-ov-file#readme) of PX4 and Gazebo Sim
	- Did this before using the colcon build comand
	- Downgrade empy python template
```bash
pip3 uninstall empy
pip3 install empy==3.3.4
```
2. After installation, possibly add QGC (QGrtoundControl)
```Shell
cd
sudo usermod -a -G dialout $USER
sudo apt-get remove modemmanager -y
sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt install libqt5gui5 -y
sudo apt install libfuse2 -y
sudo apt install wget -y
wget https://d176tv9ibo4jno.cloudfront.net/latest/QGroundControl.AppImage -P ~/QGroundControl.AppImage
cd QGroundControl.AppImage
chmod +x ./QGroundControl.AppImage
```
3. There are 2 Ways of launching Gazebo:
   
3. 1. Launching Gazebo and PX4 (QGC is not launched)([This example has been designed to run from one launch file that will start all the necessary nodes. The launch file will run a python script that uses gnome terminal to open a new terminal window for MicroDDS and Gazebo](https://github.com/ARK-Electronics/ROS2_PX4_Offboard_Example?tab=readme-ov-file#readme))(Read website to see what nodes are started, etc):
```bash
cd
cd ros2_px4_workspace/
source install/setup.bash
ros2 launch px4_offboard offboard_velocity_control.launch.py
```
- NOTE: Rebuild px4_offboard every time we make changes to the code
```Shell
cd
cd ros2_px4_workspace/
colcon build --packages-select px4_offboard
source install/setup.bash
ros2 launch px4_offboard offboard_velocity_control.launch.py
```

3. 2. Opening Simulation Files individually (Might be useful when changing worlds and models) (QGC can optionally also be launched, if installed) ([Git repo](https://github.com/nikhilsnayak3473/ROS2-PX4))([Setup video from developer, briefly diving into code](https://www.youtube.com/watch?v=8gKIP0OqHdQ)):

- Open a new terminal and run MicroXRCEAgent:
```Shell
cd
MicroXRCEAgent udp4 -p 8888
```
-  In another terminal run QGC:(Only if installed)
```Shell
cd
cd QGroundControl.AppImage
./QGroundControl.AppImage 
```
-  In another terminal start gz_x500 simulated drone:
```Shell
cd
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```
3. 2.1. Changing Models (Select from standard repo:)
- Replace "x500" with other model, e.g.
```Shell
cd
cd ~/PX4-Autopilot
make px4_sitl gz_advanced_plane
```
3. 2.2. Changing Worlds (Select from standard repo:)
- Add world name after model name, e.g.
```Shell
cd
cd ~/PX4-Autopilot
make px4_sitl gz_advanced_plane_baylands
```
	OR USE:
```Shell
cd
cd ~/PX4-Autopilot
PX4_GZ_WORLD=baylands make px4_sitl gz_x500
```
3. 2.3. Additional startup syntax available [here](https://docs.px4.io/main/en/sim_gazebo_gz/):
replace/add to "PX4_GZ_WORLD=baylands" to change other simulation parameters




# Custom worlds and models

1. Custom Worlds (.sdf files)
- Available [here](https://app.gazebosim.org/fuel)
- Paste the .sdf file into "/home/ross/PX4-Autopilot/Tools/simulation/gz/worlds"
- Run world using (e.g. baylands world)
```Shell
cd
cd ~/PX4-Autopilot
PX4_GZ_WORLD=baylands make px4_sitl gz_x500
```

2. Custom Model
- d
- Run model using (e.g. )
```Shell
cd
cd ~/PX4-Autopilot
PX4_SIM_MODEL=gz_x500 make px4_sitl 
```

	FOLLOWING NOT WORKING YET!!!!!
```Shell
cd
cd ~/PX4-Autopilot
PX4_SYS_AUTOSTART=4001 PX4_SIM_MODEL=gz_cerberus_anymal_b_visual_only ./build/px4_sitl_default/bin/px4
```




# To Change environment in Launch file and include QGC in startup

1. Open SRC from VSCode. Located in following directory:
```Shell
/home/ross/ros2_px4_workspace/src
```

2. navigate to following directory:
```Shell
wn/home/ross/ros2_px4_workspace/src/ROS2_PX4_Offboard_Example/px4_offboard/px4_offboard/processes.py
```

3. Include QGC in launch(If QGC installed): Change commands section as follows:
	1. add "," after PX4 SITL run command
	2. Ensure correct directory for QGC. NOTE how QGroundControl.AppImage is a file inside thes QGroundControl.AppImage folder.
```Shell
commands = [
# Run the Micro XRCE-DDS Agent
"MicroXRCEAgent udp4 -p 8888",

# Run the PX4 SITL simulation
"cd && cd ~/PX4-Autopilot && PX4_GZ_WORLD=default make px4_sitl gz_x500",

# Run QGroundControl
"cd && cd QGroundControl.AppImage && ./QGroundControl.AppImage"
]
```

4. Changing the model:
	1. Simply change the "x500" model in the PX4 SITL run command to another model in the models directory(Described above). e.g.:
```Shell
# Run the PX4 SITL simulation
"cd ~/PX4-Autopilot && make px4_sitl gz_advanced_plane",
```

5. Change environment:
	1. NB!!
	2. This is quite difficult
	3. I ended up changing the "default world" by:
		1. Including packages for SubT world (The links) after "/spherical_coordinates>"
		2. Removing the "model name='ground_plane'>"
	4. What you cant do for some reason(made me struggle a lot)
		1. DO NOT rename the world file

6. Change Initial Position of Drone
	1. The position at the beginning of the SubT circuit is around: '-13,0,0,0,0,0'
```Shell
# Run the PX4 SITL simulation
"cd && cd ~/PX4-Autopilot && PX4_GZ_WORLD=default PX4_GZ_MODEL_POSE='-13,0,0,0,0,0' PX4_SIM_MODEL=gz_x500_depth ./build/px4_sitl_default/bin/px4",
```

7. REMEMBER: Rebuild px4_offboard every time we make changes to the code. Use the following command for rebuild and launch:
```Shell
cd
cd ros2_px4_workspace/
colcon build --packages-select px4_offboard
source install/setup.bash
ros2 launch px4_offboard offboard_velocity_control.launch.py
```






# To Add bridge for point cloud visualization

Either open new terminal and run
```shell
ros2 run ros_gz_bridge parameter_bridge --ros-args -p config_file:=/home/nicoepler/ros2_px4_workspace/src/ROS2_PX4_Offboard_Example/px4_offboard/resource/ros_gz_bridge.yaml
```


Or from launch file:

1. Add following node to launch file:
```shell
# Bridge ROS topics and Gazebo messages for establishing communication
bridge = Node(
package='ros_gz_bridge',
executable='parameter_bridge',
parameters=[{
'config_file': os.path.join(package_dir, 'ros_gz_bridge.yaml'),
'qos_overrides./tf_static.publisher.durability': 'transient_local',
}],
output='screen'
)
```

2. Add ros_bridge_yaml file under  "/home/nicoepler/ros2_px4_workspace/src/ROS2_PX4_Offboard_Example/px4_offboard/resource/ros_gz_bridge.yaml"
3. Add the following to package.xml file:
```shell
<depend>ros_gz</depend>
<depend>ros_gz_bridge</depend>
```
4. Install ros_gz dependenceies following [this link](https://github.com/gazebosim/ros_gz/tree/humble)
5. And source workspace in "gedit ~/.bashrc" file using following line of code:
```Shell
source /home/nicoepler/ros_gz_bridge_ws/install/setup.bash
```






Troubleshooting:
1. If you get launch error "ERROR gz_bridge] Service call timed out." Check out [Link 1](https://github.com/PX4/PX4-Autopilot/issues/20668) and [Link 2](https://github.com/PX4/PX4-Autopilot/issues/22148)
2. Sometimes Gazebo struggles to run/fails. Then close VSCode. It sometimes causes problems






# Launch Command

7. REMEMBER: Rebuild px4_offboard every time we make changes to the code. Use the following command for rebuild and launch:
```Shell
cd
cd ros2_px4_workspace/
colcon build --packages-select px4_offboard
source install/setup.bash
ros2 launch px4_offboard offboard_velocity_control.launch.py
```

ros2 run tf2_tools view_frames


# RVIZ

1. Fixed frame must be set to to see pointcloud
```Shell
x500_depth_0/OakD-Lite/base_link/StereoOV7251
```
2. Adding Fixed Frames to RVIZ2 (adding world and x500_depth_0/OakD-Lite/base_link/StereoOV7251)
```Shell
ros2 run tf2_ros static_transform_publisher 0 0 0 0 0 0 1 map world
ros2 run tf2_ros static_transform_publisher 0 0 0 0 0 0 1 world x500_depth_0/OakD-Lite/base_link/StereoOV7251
```

-maybe add joint_state_publisher
-Try spawning model in launch file



















4. Uninstall gazebo harmonic using [this link](https://gazebosim.org/docs/harmonic/install_ubuntu)
5. Install gazebo garden using [this link](https://gazebosim.org/docs/garden/install_ubuntu)
6. Run the following line from [here](https://docs.px4.io/v1.14/en/sim_gazebo_gz/), else somehow the launch file fails. IDK why.:
```Shell
cd /path/to/PX4-Autopilot
make px4_sitl gz_x500
```
11. Then Run simulation using garden using:
```Shell
cd
cd ros2_px4_workspace/
colcon build --packages-select px4_offboard
source install/setup.bash
ros2 launch px4_offboard offboard_velocity_control.launch.py
```
12. Watch the [video](https://www.youtube.com/watch?v=DsjJtC8QTQY) for bridge between gazebo garden and ros2 humble








