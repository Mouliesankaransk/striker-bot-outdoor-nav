# striker-bot-outdoor-nav
ROS2-based GPS Navigation System for Outdoor Robots - Complete hardware package with u-blox F9P, RealSense D456i, ESP32, and Jetson Orin Nano.

# Striker Bot Hardware Package

Hardware interface package for Striker Bot with GPS navigation.

## Hardware Setup

### Components:
1. **Jetson Orin Nano** - Main computer
2. **u-blox F9P GPS** - High-precision GPS module
3. **RealSense D456i** - IMU source
4. **ESP32** - Motor controller
5. **LiDAR** (optional) - For obstacle detection

### Connections:
- GPS: `/dev/ttyACM0` (USB)
- ESP32: `/dev/ttyUSB0` (USB)
- RealSense: USB 3.0

## Installation

1. **Clone and build:**
mkdir -p ~/striker_hardware_ws/src
cd ~/striker_hardware_ws/src
git clone <repository>
cd ~/striker_hardware_ws
colcon build --packages-select striker_bot_hardware
source install/setup.bash

2. **Install dependencies:**
sudo apt install ros-humble-robot-localization ros-humble-realsense2-camera
pip3 install pyserial pynmea2


3. **commands:**
    1. Launch complete system:
        ros2 launch striker_bot_hardware hardware.launch.py

    2. Record GPS waypoints:
        ros2 run striker_bot_hardware gps_waypoint_logger

    3. Navigate to saved waypoints:
        ros2 run striker_bot_hardware gps_navigator ~/waypoints.yaml

    4. Test GPS:
        ros2 run striker_bot_hardware test_gps_basic


5. **Troubleshooting**

    GPS not detected:

        Check permissions: sudo chmod 666 /dev/ttyACM0

        Check connection: ls /dev/ttyACM*

    ESP32 not responding:

        Check baudrate in motor_controller.py

        Verify ESP32 firmware

    Navigation fails:

        Check TF frames: ros2 run tf2_ros tf2_echo map base_link

        Verify GPS fix quality



## **Deployment Script:**

Create `deploy_to_jetson.sh`:


#!/bin/bash
# Deploy script for Jetson Orin Nano

JETSON_IP="192.168.1.100"  # Change to your Jetson IP
JETSON_USER="nvidia"       # Change to your username

echo "Deploying to Jetson Orin Nano..."

# 1. Create archive
        tar -czf striker_hardware.tar.gz striker_hardware_ws/

# 2. Copy to Jetson
        echo "Copying to Jetson..."
        scp striker_hardware.tar.gz ${JETSON_USER}@${JETSON_IP}:~/

# 3. Install commands
        echo "Installing on Jetson..."
        ssh ${JETSON_USER}@${JETSON_IP} << 'EOF'
        cd ~
        tar -xzf striker_hardware.tar.gz
        cd striker_hardware_ws
        colcon build --packages-select striker_bot_hardware
        echo "source ~/striker_hardware_ws/install/setup.bash" >> ~/.bashrc
        EOF

        echo "Deployment complete!

6. **To Deploy:**

    1. On your development machine:

        # Create the structure
        mkdir -p ~/striker_hardware_ws/src/striker_bot_hardware
        # Copy all the files above into their respective locations

        # Create deploy script
        chmod +x deploy_to_jetson.sh
        ./deploy_to_jetson.sh

    2. On Jetson Orin Nano:

        # Test the system
        source ~/striker_hardware_ws/install/setup.bash

        # Test GPS
        ros2 run striker_bot_hardware test_gps_basic

        # Launch full system
        ros2 launch striker_bot_hardware hardware.launch.py

