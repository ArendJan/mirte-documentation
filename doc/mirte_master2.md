# Mirte Master


The Mirte Master is one of the possible upgrades of the Mirte robot.

<!-- .. plaatje -->

## Frame


The frame is built using 1mm aluminium and 3D printed parts. 
<!-- .. TODO: add partslist and 3D print files -->


### Base

The base:

<!-- .. plaatje -->


### Arm

The arm has 4 servos for movement and one for the gripper.

+----------+----+-----------+------+
| Servo    | ID | Range     | Home |
+==========+====+===========+======+
| Rotation | 2  | 1000-2000 | 1500 |
+----------+----+-----------+------+
| Shoulder | 3  |           |      |
+----------+----+-----------+------+
| Elbow    | 4  |           |      |
+----------+----+-----------+------+
| Wrist    | 5  |           |      |
+----------+----+-----------+------+
| Gripper  | 6  |           |      |
+----------+----+-----------+------+

#### Moveit


<!-- .. TODO: @mklomp -->


## Software


### Computer

The computer is a Orange Pi 3B with 4gb RAM and 16/32 eMMC. It is running a Armbian 23.11 Focal image with ROS Noetic. You can download a fresh image from `here <https://github.com/ArendJan/mirte-sd-image-tools/actions/workflows/buildFork.yaml>`_. Click on the latest action and then download the ```mirte_master_mirte_orangepi3b``` or ```mirte_master_installer_orangepi3b``` artifact.


# SW


## Mirte-sw

### Master Mirte-install-scripts
[Repo](https://github.com/ArendJan/mirte-install-scripts/tree/mirte-master2)

Changes from default:
- Password system:
  - After changing the pw (first login):
    - the user password changes
    - the website login changes
    - after the next boot: the Wi-Fi password is changed
- Website:
  - different urls:
    - video server: http://IP/ros-video/
    - jupyter (if enabled?): http://IP/jupyter/
    - docs: http://IP/docs/
    - visual studio code (browser): http://IP/code/
  - Login:
    - When accessing any of the sites from any IP other 192.168.40.1-47.255 (Wi-Fi hotspot) or 192.168.137.1-255 (Windows hotspot), you'll need to log in. This is the same user and password as when logging in with SSH (mirte:<your pw>)
    - To allow more IP ranges, edit ```/etc/nginx/nginx_login.conf``` 
- extra ROS packages:
  - https://github.com/Slamtec/rplidar_ros.git: lidar node
  - https://github.com/orbbec/ros_astra_camera.git: depth camera node
  - https://github.com/arendjan/ridgeback.git: mecanum wheel support with custom fix.
- Picotool installed to easily flash the Raspberry Pi Pico
- more ZSH support
- Support for @chris-pek his chat-gpt ROS node
- USB-switch systemctl module:
  - after boot, this module will turn on a GPIO pin (GPIO4_C3) to turn on the USB switch IC on the small board on top of the Orange Pi. This is to fix a power bug of the Orbbec Astra camera (pulling 1A+ when plugged in at boot)

### Mirte-ros-packages
[Repo](https://github.com/ArendJan/mirte-ros-packages/tree/mirte-master)

Changes:
- Added extra launch files (mirte_bringup)
- added omniwheels (ridgeback) support (mirte_control)
  - TODO: encoder
- Telemetrix:
  - Extra config for mirte-master
    - TODO: check arm and gripper angles, GPIO SOC led
  - Oled: shows SOC, IP, hostname and Wi-Fi host
  - Hiwonder UART servos:
    - Added servo support, services/topics:
      - ```/mirte/set_{servo_name}_servo_angle``` SetServoAngle service, in radians from home
      - ```/mirte/set_{servo_name}_servo_enable``` SetBool service, enable or disable servo. After sending a new angle, it is enabled again
      - ```/mirte/servos/{servo_name}/position``` ServoPosition topic, radians from home
      - ```/mirte/set_{bus_name}_all_servos_enable``` SetBool service, enable or disable all servos on this bus.
  - Neopixel support (WS2811)
    - ```/mirte/set_{name}_color_all``` SetLedValue service, uint32_t color, with ```0xRRGGBB``` or ```0xRRBBGG``` value (0-255)
    - ```/mirte/set_{name}_color_single``` SetSingleLedValue service, same color setup with a 0-indexed pixel id
  - PCA9685 (pwm driver):
    - For servos:
      - When any servo is added, frequency will be set to 50Hz. 
      - ```/mirte/set_{name}_servo_angle``` SetServoAngle service, with angle [0, 180]
    - Motors (2x PWM channel):
      - Normal motor service and topics
      - Range is [-100, 100]
  - INA226 (power watcher)
    - used to measure current and voltage. Can trigger a shutdown if the LiPo voltage is too low. The current state of charge(SOC) can be shown with a LED and the OLED screen.
      Also detects the power switch to safely shut down the robot.

