# uav_waypoint_navigation
This is a **ROS 2 Humble + Gazebo Classic** simulation-only autonomous UAV: kinematic-controlled quadrotor with waypoint navigation, LiDAR-based reactive obstacle avoidance, return-to-home, and emergency stop (no PX4/flight controller).

## Setup

```bash
sudo apt install ros-humble-cv-bridge python3-opencv
# plus the usual: ros-humble-gazebo-ros-pkgs, robot-state-publisher, xacro, rviz2
```

## Quick start

```bash
cd uav_ws
bash run.sh              # first run builds with colcon, then launches Gazebo + RViz2
bash run.sh --clean      # force a full rebuild
```
Wait ~10s for Gazebo and RViz2 to appear.

## Control the mission (in a second terminal)

```bash
bash mission_control.sh start      # takeoff -> waypoints -> RTH -> land
bash mission_control.sh status     # print current MissionStatus
bash mission_control.sh emergency  # manually trigger emergency stop (demo)
bash mission_control.sh clear      # clear the emergency latch
bash mission_control.sh rth        # abort straight to return-to-home
```

## Or call the underlying services/topics directly

```bash
ros2 service call /uav/start_mission std_srvs/srv/Trigger {}
ros2 service call /uav/takeoff std_srvs/srv/Trigger {}
ros2 service call /uav/land std_srvs/srv/Trigger {}
ros2 service call /uav/return_home std_srvs/srv/Trigger {}
ros2 service call /uav/trigger_emergency std_srvs/srv/Trigger {}
ros2 service call /uav/clear_emergency std_srvs/srv/Trigger {}

ros2 topic echo /uav/mission_status
ros2 topic echo /uav/battery_state
ros2 topic echo /uav/obstacle_status
```

## Manual build (equivalent to what run.sh does)

```bash
cd uav_ws
source /opt/ros/humble/setup.bash
colcon build --symlink-install
source install/setup.bash
export LIBGL_ALWAYS_SOFTWARE=1
ros2 launch uav_bringup uav_sim.launch.py
```

## Editing the mission

Edit `src/uav_bringup/config/waypoints.yaml` (`waypoints_x/y/z` under `mission_manager`) — local ENU meters. The default mission flies past 5 obstacles so you can watch reactive avoidance trigger near waypoints 2 and 3.

## Troubleshooting

```bash
# If the UAV doesn't move, check the controller is alive:
ros2 service list | grep goto
ros2 topic hz /uav/odom

# Full clean rebuild if things get weird:
rm -rf build/ install/ log/ && colcon build --symlink-install
```
- If `/gazebo/set_entity_state` never appears, check Gazebo's console for `libgazebo_ros_state.so` load errors and confirm `ros-humble-gazebo-ros-pkgs` is installed.
- RViz Fixed Frame is `map` — the controller broadcasts `map -> base_link` directly (no separate odom frame; this is ground-truth kinematic sim, not a localization stack).

Note: the zip bundles a pre-built `build/`/`install/`/`log/` with absolute paths from `/home/aa/Downloads/uav_ws_new/...` — `run.sh` rebuilds automatically, so no need to reuse those directly. Needs a real Ubuntu 22.04 + ROS 2 Humble + Gazebo machine — won't run in this sandbox.
