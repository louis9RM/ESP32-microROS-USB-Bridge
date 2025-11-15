# 🚀 ESP32 micro-ROS + USB + ROS2 + Docker + Windows

Este proyecto documenta cómo conectar un **ESP32 con micro-ROS** usando conexión **USB (NO WiFi)** a **ROS2 Humble**, ejecutado dentro de un contenedor Docker en Windows.

La finalidad es disponer de una guía clara y funcional para que cualquiera pueda replicar el entorno sin errores.

---

## 📌 Requisitos

| Componente | Necesario |
|-----------|-----------|
| Windows 10/11 | ✔ |
| WSL2 | ✔ |
| Ubuntu 22.04 en WSL2 | ✔ |
| Docker Desktop | ✔ |
| usbipd-win | ✔ |
| Arduino IDE con librería `micro_ros_arduino` | ✔ |
| ESP32 conectado por USB | ✔ |

---

## 🧩 Paso 1 — abrir Ubuntu 22.04 en WSL2

En **PowerShell** ejecuta:

```powershell
wsl -d Ubuntu-22.04
```

No cierres esta ventana.

Dentro de Ubuntu, verifica que reconoce puertos:

```bash
ls /dev/tty*
```

Ahora vuelve a PowerShell y lista los dispositivos USB:

```powershell
usbipd list
```

Identifica tu ESP32 (ejemplo `1-3`) y ejecútalo:

```powershell
usbipd attach --wsl --busid 1-3
```

Luego vuelve a Ubuntu y valida:

```bash
ls /dev/ttyUSB*
```

✔ Si aparece `/dev/ttyUSB0`, todo está correcto.

---

## 🐳 Paso 2 — ejecutar Docker con acceso al puerto USB

En PowerShell:

```powershell
docker run -it --name ros2_dev --privileged --device=/dev/ttyUSB0 ros:humble-ros-base bash
```

Esto abre una terminal dentro del contenedor.

---

## 🏗 Paso 3 — instalar micro-ROS Agent

Ejecuta dentro del contenedor:

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

---

## 🔌 Paso 4 — ejecutar micro-ROS Agent por USB

```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0 -v6
```

---

## 📡 Paso 5 — verificar tópicos ROS2

Abrir otro terminal desde Windows:

```powershell
docker exec -it ros2_dev bash
```

Dentro del contenedor:

```bash
source /opt/ros/humble/setup.bash
source ~/microros_ws/install/setup.bash
ros2 topic list
```

Si todo funcionó deberías ver algo como:

```
/esp/int_pub
/esp/int16array_pub
/esp/led
/parameter_events
/rosout
```

Y puedes probar recibir:

```bash
ros2 topic echo /esp/int_pub
```

---

## 🔁 ¿Qué hacer después de reiniciar la PC?

Cada vez que vuelvas:

1. Conectar el ESP32 por USB  
2. Abrir PowerShell:

```powershell
usbipd attach --wsl --busid 1-3
```

3. Abrir el contenedor:

```powershell
docker exec -it ros2_dev bash
```

4. En el contenedor:

```bash
source /opt/ros/humble/setup.bash
source ~/microros_ws/install/setup.bash
ros2 topic list
```

Opcional: volver a ejecutar el agente si lo necesitas:

```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0 -v6
```

---

## ✅ Estado final del sistema

Si al ejecutar:

```
ros2 topic echo /esp/int_pub
```

recibes valores como:

```
data: 9165
data: 9166
data: 9167
```

👉 significa que ROS2 está recibiendo datos desde el ESP32 correctamente.

---

## 🏁 Conclusión

Este setup permite trabajar con **micro-ROS + ESP32 + USB** sin depender de Wi-Fi, con un entorno reproducible gracias a Docker y WSL2.

---

📌 **Autor:** *Tu Nombre*  
📌 **Versión:** 1.0  
📌 **Licencia:** MIT

