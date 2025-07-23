Se supone un entorno vacio de Ubuntu 22.04 Jammy, la version de ROS que se utilizara es ROS2 Humble.

En primer lugar se debe configurar un entorno de desarrollo PX4

cd
git clone https://github.com/PX4/PX4-Autopilot.git --recursive
bash ./PX4-Autopilot/Tools/setup/ubuntu.sh
cd PX4-Autopilot/
make px4_sitl
Instalacion de ROS 2 Humble, incluye la version de gazebo compatibleç

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
sudo apt update && sudo apt upgrade -y
sudo apt install ros-humble-desktop
sudo apt install ros-dev-tools
source /opt/ros/humble/setup.bash && echo "source /opt/ros/humble/setup.bash" >> .bashrc
Dependencias de Python

pip install --user -U empy==3.3.4 pyros-genmsg setuptools

Correr simulacion base de PX4

make px4_sitl gz_x500

Instalacion y creacion de workspace MAVROS para ROS2

En nuestro directorio raiz:

mkdir -p ~/mavros_ws/src
cd ~/mavros_ws
sudo apt update
sudo apt install -y python3-vcstool python3-rosinstall-generator python3-osrf-pycommon

Generar archivos .repo para Mavlink y MAVROS

rosinstall_generator --format repos mavlink | tee /tmp/mavlink.repos
rosinstall_generator --format repos --upstream mavros | tee -a /tmp/mavros.repos

Descargar codigo fuente usando VCS

vcs import src < /tmp/mavlink.repos
vcs import src < /tmp/mavros.repos
Instalar dependencias

rosdep install --from-paths src --ignore-src -y

Instalar datos de geographiclib para mavros gps

sudo ./src/mavros/mavros/scripts/install_geographiclib_datasets.sh

Compilar Workspace

colcon build –symlink-install

Correr Mavros

ros2 launch mavros px4.launch fcu_url:=udp://:14540@localhost:14557

Instalacion QGROUNDCONTROL

En una terminal externa:

sudo usermod -aG dialout "$(id -un)"
sudo systemctl mask --now ModemManager.service
sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-libav gstreamer1.0-gl -y
sudo apt install libfuse2 -y
sudo apt install libxcb-xinerama0 libxkbcommon-x11-0 libxcb-cursor-dev -y
ir a https://docs.qgroundcontrol.com/master/en/qgc-user-guide/getting_started/download_and_install.html

Descargar el archivo: QgroundControl-x86_64.AppImage.

En la terminal, accedemos a la ruta donde esta el archivo .appimage y otorgamos permiso de ejecucion

chmod +x QgroundControl-x86

PROCEDIMIENTO PARA EJECUTAR UNA SIMULACION BASICA

1. ejecutamos QgroundControl, podemos hacer doble click en el archivo.
2. Aparecera desconectado, para enlazarlo con el dron, en una terminal ingresamos al directorio PX4-Autopilot y ejecutamos: make px4_sitl gz_x500. Luego de inicializarse el PX4 se comunicara con Qgroundcontrol mediante el puerto de enlace. A la par podemos visualizar que se lanza la simulacion en gazebo.
3.Para enlazar el entorno de Mavros simulado en una terminal sourceda con el paquete mavros_ws ejecutamos: ros2 launch mavros px4.launch fcu_url:=udp://:14540@localhost:14557
4. Podemos visualizar los mensajes de conexion tanto desde la terminal de mavros como en la terminal interactiva de PX4, de todas maneras para comprobar la comunicacion podemos ejecutar el comando: ros2 topic echo /mavros/state. Este comando nos mostrara la configuracion del dron y el estado conectado.
5. Finalmente para armar el dron debemos setear un setpoint lo hacemos en otra terminal adicional ejecutando: ros2 topipub -r 10 /mavros/setpoint_position/local geometry_msgs/msg/PoseStamped "header:
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
6. Ahora el dron tiene un setpoint, por lo que podemos armarlo ya sea desde la terminal interactiva PX4 , Qground o mavros. Ahora lo haremos con mavros usando el comando:
ros2 service call /mavros/set_mode mavros_msgs/srv/SetMode "{custom_mode: 'OFFBOARD'}" cambia a modo offboard. Y el comando: ros2 service call /mavros/cmd/arming mavros_msgs/srv/CommandBool "{value: true}" para armar el dron y que se dirija al setpoint enviado anteriormente.
7. Todo listo, ahora se pueden implementar nodos para controlar el dron usando mavros o incluso enviarle trayectorias a seguir directamente desde qground.
