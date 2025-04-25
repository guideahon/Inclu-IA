# Inclu-IA
Proyecto de subtitulado en tiempo real con IA, Raspberry Pi 4 y micrófono Bluetooth

---

## Índice

- [Descripción](#descripción)  
- [Nombres propuestos](#nombres-propuestos)  
- [Características](#características)  
- [Requisitos](#requisitos)  
  - [Hardware](#hardware)  
  - [Software](#software)  
- [Estructura del repositorio](#estructura-del-repositorio)  
- [Instalación](#instalación)  
  - [1. Actualizar Raspbian](#1-actualizar-raspbian)  
  - [2. Configurar punto de acceso Wi-Fi](#2-configurar-punto-de-acceso-wi-fi)  
  - [3. Emparejar micrófono Bluetooth](#3-emparejar-micrófono-bluetooth)  
  - [4. Instalar fast-whisper.cpp](#4-instalar-fast-whispercpp)  
- [Uso](#uso)  
  - [A. Iniciar servicios de red](#a-iniciar-servicios-de-red)  
  - [B. Ejecutar el servidor de transcripción](#b-ejecutar-el-servidor-de-transcripción)  
  - [C. Acceder a la interfaz web](#c-acceder-a-la-interfaz-web)  
- [Optimización y troubleshooting](#optimización-y-troubleshooting)  
- [Contribuciones](#contribuciones)  
- [Licencia](#licencia)  

---

## Descripción

Inclu-IA es un sistema de subtitulado en tiempo real para aulas, basado en Raspberry Pi 4 y un micrófono Bluetooth, que captura la voz del profesor, la transcribe usando IA (fast-whisper.cpp) y la emite vía un punto de acceso Wi-Fi a una página web de subtítulos para estudiantes con discapacidad auditiva.


---

## Características

- Captura de audio inalámbrica vía Bluetooth  
- Transcripción en streaming con fast-whisper.cpp  
- Punto de acceso Wi-Fi integrado (modo AP)  
- Cliente web ligero con subtítulos en tiempo real  
- Fácil escalabilidad cambiando modelos de IA  

---

## Requisitos

### Hardware

- **Raspberry Pi 4** (conraspbian 64-bit)  
- **Micrófono Bluetooth** (p.ej. BoomArm BT-800)  
- **Tarjeta SD** ≥ 16 GB (clase 10)  
- Fuente de alimentación estable  

### Software

- **Sistema operativo**: Raspbian (actualizado)  
- **Paquetes básicos**:  
  - `git`, `build-essential`, `python3-pip`  
  - `hostapd`, `dnsmasq`, `avahi-daemon`  
- **Audio/Bluetooth**:  
  - `pulseaudio`, `pulseaudio-module-bluetooth`, `bluez-tools`  
  - `python3-pyaudio`  
- **Servidor y streaming**:  
  - Python 3: `Flask`, `Flask-SocketIO`, `python-socketio`, `PyAudio`  




PASOS:

```markdown
## Instalación

1. **Actualizar Raspbian**  
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo reboot
   ```  
   Instala herramientas básicas:  
   ```bash
   sudo apt install -y git build-essential python3-pip \
     hostapd dnsmasq avahi-daemon
   ```

2. **Configurar punto de acceso Wi-Fi**  
   - Editar `/etc/hostapd/hostapd.conf`:  
     ```ini
     interface=wlan0
     ssid=Subtitulos_AP
     hw_mode=g
     channel=6
     auth_algs=1
     wpa=2
     wpa_passphrase=TuClaveSegura
     wpa_key_mgmt=WPA-PSK
     wpa_pairwise=TKIP
     rsn_pairwise=CCMP
     ```
   - Apuntar al archivo en `/etc/default/hostapd`:  
     ```bash
     DAEMON_CONF="/etc/hostapd/hostapd.conf"
     ```
   - Configurar DHCP en `/etc/dnsmasq.conf`:  
     ```ini
     interface=wlan0
     dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
     ```
   - Asignar IP estática en `/etc/dhcpcd.conf`:  
     ```ini
     interface wlan0
       static ip_address=192.168.4.1/24
       nohook wpa_supplicant
     ```
   - Habilitar y arrancar:  
     ```bash
     sudo systemctl enable hostapd dnsmasq
     sudo systemctl start hostapd dnsmasq
     ```

3. **Emparejar micrófono Bluetooth**  
   ```bash
   sudo apt install -y pulseaudio pulseaudio-module-bluetooth bluez-tools python3-pyaudio
   ```  
   Editar `/etc/pulse/system.pa` y añadir:  
   ```ini
   load-module module-bluetooth-policy
   load-module module-bluetooth-discover
   ```  
   Luego:  
   ```bash
   bluetoothctl
   # dentro de bluetoothctl:
   scan on
   pair XX:XX:XX:XX:XX:XX
   trust XX:XX:XX:XX:XX:XX
   connect XX:XX:XX:XX:XX:XX
   exit
   ```  
   Verifica el dispositivo:  
   ```bash
   pactl list sources short
   ```

4. **Instalar fast-whisper.cpp**  
   ```bash
   git clone https://github.com/ggerganov/whisper.cpp
   cd whisper.cpp
   make
   ```  
   Descarga un modelo (por ejemplo, “tiny”):  
   ```bash
   ./models/download-ggml-model.sh tiny.en
   ```  
   Prueba con un archivo de audio:  
   ```bash
   arecord -f S16_LE -r 16000 -d 5 test.wav
   ./main -m models/ggml-tiny.en.bin -f test.wav
   ```

## Uso

### A. Iniciar servicios de red
```bash
sudo systemctl start hostapd dnsmasq
```

### B. Ejecutar el servidor de transcripción
```bash
cd software
pip3 install -r requirements.txt
python3 server.py
```

### C. Acceder a la interfaz web
1. Conéctate a la red **Subtitulos_AP** (clave: `TuClaveSegura`).  
2. Abre el navegador en `http://192.168.4.1:5000`.  
   Verás los subtítulos en tiempo real en pantalla.

## Optimización y troubleshooting

- **Modelos**: prueba `tiny` → `small` → `medium` según potencia y latencia.  
- **Ganancia**: ajusta la ganancia en PulseAudio si hay mucho ruido.  
- **Buffer**: modifica `CHUNK` en `server.py` para equilibrar latencia y estabilidad.  
- **Logs**: activa verbose en Flask o fast-whisper para diagnosticar errores.

## Contribuciones

1. Haz un **fork** del repositorio.  
2. Crea una rama para tu feature (`git checkout -b feature/X`).  
3. Realiza **commits** claros y descriptivos.  
4. Abre un **pull request** describiendo tus cambios.
```
