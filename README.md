# ESP32 micro-ROS con USB + ROS2 + Docker + Windows

Este documento describe cómo conectar un **ESP32 con micro-ROS** a **ROS2 Humble** ejecutándose en **Docker en Windows**, usando **USB (NO WiFi)**.

---

## 🚀 Requisitos

- Windows 10/11
- WSL2
- Ubuntu 22.04 instalado en WSL2
- Docker Desktop
- usbipd-win instalado
- Arduino IDE con librería micro-ROS Arduino
- ESP32 con cable USB

---

##  abrir Ubuntu 22.04

Ejecuta esto en PowerShell:

```powershell
wsl -d Ubuntu-22.04

```

Esto abrirá la terminal Linux.

Déjala abierta (esto es importante).

Cuando estés dentro, ejecuta dentro de Ubuntu:

```
ls /dev/tty*

```
Luego, SIN cerrar esa ventana, vuelve a PowerShell y ejecuta:


```powershell
usbipd attach --wsl --busid 1-3

```
Después vuelve a Ubuntu y ejecuta:

```powershell
ls /dev/ttyUSB*

```
Si todo está correcto deberías ver:

/dev/ttyUSB0


##  Ejecutar el contenedor Docker con acceso al puerto

En PowerShell, ejecuta:

```bash
docker run -it --name ros2_dev --privileged --device=/dev/ttyUSB0 ros:humble-ros-base bash

```
Esto abrirá una terminal dentro del contenedor.

## Instalar micro-ROS Agent
```bash
apt update && apt install -y git python3-pip build-essential
pip3 install -U colcon-common-extensions

mkdir -p ~/microros_ws/src
cd ~/microros_ws/src
git clone -b humble https://github.com/micro-ROS/micro-ROS-Agent.git
git clone -b humble https://github.com/micro-ROS/micro_ros_msgs.git

cd ~/microros_ws
source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash
```

## Ejecutar el micro-ROS Agent por USB
```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0 -v6
```

## Verificar los tópicos en otra terminal
```bash
docker exec -it ros2_dev bash
```
```bash
source /opt/ros/humble/setup.bash
source ~/microros_ws/install/setup.bash
ros2 topic list
```


