# Guía de Simulación PX4 + MAVROS + QGroundControl

Esta guía explica cómo instalar y ejecutar una simulación completa de dron usando PX4, ROS 2 Humble, Gazebo y MAVROS sobre Ubuntu 22.04.

---

## 1️⃣ Instalación de PX4

```bash
cd ~
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
cd PX4-Autopilot/
make px4_sitl
```

---

## 2️⃣ Instalación de ROS 2 Humble + Gazebo

```bash
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y

sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | \
sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update && sudo apt upgrade -y
sudo apt install ros-humble-desktop
sudo apt install ros-dev-tools

source /opt/ros/humble/setup.bash
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

### ➕ Dependencias Python

```bash
pip install --user -U empy==3.3.4 pyros-genmsg setuptools
```

---

## 3️⃣ Simulación PX4 en Gazebo

```bash
make px4_sitl gz_x500
```

---

## 4️⃣ Crear workspace para MAVROS

```bash
mkdir -p ~/mavros_ws/src
cd ~/mavros_ws

sudo apt update
sudo apt install -y python3-vcstool python3-rosinstall-generator python3-osrf-pycommon

rosinstall_generator --format repos mavlink | tee /tmp/mavlink.repos
rosinstall_generator --format repos --upstream mavros | tee -a /tmp/mavros.repos

vcs import src < /tmp/mavlink.repos
vcs import src < /tmp/mavros.repos

rosdep install --from-paths src --ignore-src -y

sudo ./src/mavros/mavros/scripts/install_geographiclib_datasets.sh

colcon build --symlink-install
```

---

## 5️⃣ Lanzar MAVROS

```bash
source ~/mavros_ws/install/setup.bash
ros2 launch mavros px4.launch fcu_url:=udp://:14540@localhost:14557
```

---

## 6️⃣ Instalar QGroundControl

```bash
sudo usermod -aG dialout "$(id -un)"
sudo systemctl mask --now ModemManager.service

sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt install libfuse2 -y
sudo apt install libxcb-xinerama0 libxkbcommon-x11-0 libxcb-cursor-dev -y
```

📥 Descargar desde:  
https://docs.qgroundcontrol.com/master/en/qgc-user-guide/getting_started/download_and_install.html

Luego:

```bash
chmod +x QgroundControl-x86_64.AppImage
```

---

## 🚀 Ejecución de la Simulación Completa

1. Ejecutar QGroundControl (doble clic o terminal)
2. En otra terminal:

```bash
cd ~/PX4-Autopilot
make px4_sitl gz_x500
```

3. En nueva terminal:

```bash
source ~/mavros_ws/install/setup.bash
ros2 launch mavros px4.launch fcu_url:=udp://:14540@localhost:14557
```

4. Verificar conexión:

```bash
ros2 topic echo /mavros/state
```

5. Publicar setpoint (en nueva terminal):

```bash
ros2 topic pub -r 10 /mavros/setpoint_position/local geometry_msgs/msg/PoseStamped "header:
  frame_id: 'map'
pose:
  position:
    x: 0.0
    y: 0.0
    z: 2.0
  orientation:
    x: 0.0
    y: 0.0
    z: 0.0
    w: 1.0"
```

6. Cambiar a modo OFFBOARD y armar:

```bash
ros2 service call /mavros/set_mode mavros_msgs/srv/SetMode "{custom_mode: 'OFFBOARD'}"
ros2 service call /mavros/cmd/arming mavros_msgs/srv/CommandBool "{value: true}"
```

✅ ¡Listo! Puedes enviar trayectorias desde QGroundControl o usar tus propios nodos ROS 2.
