
# nstallation and use of ROS2 Jazzy Jalisco on a Windows computer 
<br>

## table of contents 

- [Installation of Ubuntu on Windows via WSL](#installation-of-ubuntu-on-windows-via-wsl)
- [Installation of ROS2 Jazzy Jalisco on Ubuntu](#installation-of-ros2-jazzy-jalisco-on-ubuntu)
- [Installation of ur_robot_driver](#installation-of-ur-robot-driver)
- [Robot and computer configuration](#robot-and-computer-configuration)

<br>

---

## Installation of Ubuntu on Windows via WSL
- Open PowerShell :
```powershell
wsl --install
```

- Restart the computer 
- Open Ubuntu
- Create a default Unix user and a new password

<br>

---

## Installation of ROS2 Jazzy Jalisco on Ubuntu 

#### Setting language parameters
```bash
locale # Make sure lines contain “en_US.UTF-8” else: 

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
```

#### Installation of packages
```bash
sudo apt upgrade -y
sudo apt install -y curl gnupg2 lsb-release
```

#### Add ROS2 GPG key and repository
```bash
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key | sudo gpg --dearmor -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

#### Final installation of ROS2 Jazzy Jalisco
```bash
sudo apt update # Updating the list of available packages
sudo apt install ros-jazzy-desktop
source /opt/ros/jazzy/setup.bash # Environment set up
```
<br>

---

## Installation of ur_robot_driver 

##### Installation of colcon, it’s extensions and vcs
```bash
sudo apt install python3-colcon-common-extensions python3-vcstool
```

##### Creation of a worspace
```bash
export COLCON_WS=~/workspace/ros_ur_driver
mkdir -p $COLCON_WS/src
```

##### Clone the ROS 2 driver repository
```bash
cd $COLCON_WS
git clone -b jazzy https://github.com/UniversalRobots/Universal_Robots_ROS2_Driver.git src/Universal_Robots_ROS2_Driver 
```

##### Installation of dependencies
```bash
vcs import src --skip-existing --input src/Universal_Robots_ROS2_Driver/Universal_Robots_ROS2_Driver-not-released.${ROS_DISTRO}.repos
rosdep update #Updating the rosdep database
rosdep install --ignore-src --from-paths src -y
```

##### Packages compilation
```bash
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
```

##### Source the workspace 
```bash
source install/setup.bash
echo "source $COLCON_WS/install/setup.bash" >> ~/.bashrc #Automatically source the setup.bash script for each terminal opened
```

##### In case of compilation errors
```bash
cd $COLCON_WS
vcs import src --skip-existing --input src/Universal_Robots_ROS2_Driver/Universal_Robots_ROS2_Driver.${ROS_DISTRO}.repos
rosdep update
rosdep install --ignore-src --from-paths src -y
```
<br>

---

## Robot and computer configuration 
##### Setting up the Universal Robot 5.11 on the same local network as the computer

**On the robot graphic interface:**  

- Go to the menu (the three bars at top right) → Settings → System → Network  
- Check information about the network of the robot:  
<p style="padding-left: 40px;"> - Static Address is selected </p>  
<p style="padding-left: 40px;"> - IP address = 192.168.1.101</p>  
<p style="padding-left: 40px;"> - Subnet mask = 255.255.255.0</p>


**On a computer terminal**, check if the computer IP address is 
in 192.168.1.x form with the command : 
```bash
hostname -I
```

If not : Identify Ethernet interface &lt;eth0&gt; by connecting robot and computer via ethernet cable and with the command:
```bash
ip link
```

Assign an IP address <192.168.1.102> to the computer
```bash
sudo ip addr add 192.168.1.102/24 dev <eth0>
sudo ip link set dev <eth0> up
```

**On robot graphic interface :**   

- Go to Installation → URCaps → External Control   
- Set Host IP with the IP of the computer <192.168.1.102>  
- Save  

**On a terminal**, check the connexion with the robot : 
```bash
ping 192.168.1.101 #if you get an answer, the connection is right
```

Then launch the driver:
```bash
sudo apt update
sudo apt install ros-jazzy-rviz2 #Installation of rviz2

ros2 launch ur_robot_driver ur_control.launch.py \
  ur_type:=ur3e \
  robot_ip:=192.168.1.101 \
  launch_rviz:=true
```

