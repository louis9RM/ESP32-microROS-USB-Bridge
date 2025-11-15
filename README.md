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

## 1️⃣ abrir Ubuntu 22.04

Ejecuta esto en PowerShell:

Ejecutar en PowerShell:

```powershell
wsl -d Ubuntu-22.04

```

Debes ver algo como:

```
Silicon Labs CP210x USB to UART Bridge (COM6)
```

---

## 2️⃣ Ver dispositivos USB disponibles

```powershell
usbipd list
```

---

## 3️⃣ Compartir el dispositivo USB con WSL2

```powershell
usbipd bind --busid 1-3
usbipd attach --wsl --busid 1-3
```

---

## 4️⃣ Confirmar detección en Ubuntu (WSL)

```bash
ls /dev/ttyUSB*
```

Resultado esperado:

```
/dev/ttyUSB0
```

---

## 5️⃣ Crear contenedor Docker con acceso al USB

En PowerShell:

```powershell
docker run -it --name ros2_dev --privileged --device=/dev/ttyUSB0 ros:humble-ros-base bash
```

---

## 6️⃣ Instalar micro-ROS Agent dentro del contenedor

Ejecutar:

```bash
apt update && apt install -y git python3-pip build-essential
pip3 install -U colcon-common-extensions
```

Clonar micro-ROS:

```bash
mkdir -p microros_ws/src
cd microros_ws/src
git clone -b humble https://github.com/micro-ROS/micro-ROS-Agent.git
git clone -b humble https://github.com/micro-ROS/micro_ros_msgs.git
```

Compilar:

```bash
cd ~/microros_ws
source /opt/ros/humble/setup.bash
colcon build
source install/setup.bash
```

---

## 7️⃣ Ejecutar el agente micro-ROS por USB

```bash
ros2 run micro_ros_agent micro_ros_agent serial --dev /dev/ttyUSB0 -v6
```

Debes ver algo como:

```
[INFO] Serial mode agent running
[INFO] Client connected
```

---

## 8️⃣ Verificar comunicación desde otra terminal

Abrir otra terminal:

```powershell
docker exec -it ros2_dev bash
```

Luego:

```bash
source /opt/ros/humble/setup.bash
source ~/microros_ws/install/setup.bash
ros2 topic list
```

Si el ESP32 está funcionando correctamente verás tópicos.

---

## 🧪 Código mínimo para ESP32 (micro-ROS USB)

```cpp
#include <micro_ros_arduino.h>

void setup() {
  set_microros_transports(); // modo USB
  Serial.begin(115200);
}

void loop() {}
```

---

## 🛠 Problemas comunes

| Problema | Solución |
|----------|----------|
| `/dev/ttyUSB0` no aparece | Repetir `usbipd bind` + `attach` |
| micro-ROS no conecta | Confirmar `set_microros_transports()` |
| Docker no detecta dispositivo | Usar `--privileged` |

---

## ✔ Estado final

| Elemento | Estado |
|----------|--------|
| ESP32 conectado por USB | ✔ |
| micro-ROS Agent funcionando | ✔ |
| ROS2 recibiendo datos | ✔ |

---

## 📄 Licencia

MIT

---

## Autor

Configurado por **EVER**
