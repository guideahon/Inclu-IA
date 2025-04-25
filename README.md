```markdown
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
  1. [Actualizar Raspbian](#1-actualizar-raspbian)  
  2. [Configurar punto de acceso Wi-Fi](#2-configurar-punto-de-acceso-wi-fi)  
  3. [Emparejar micrófono Bluetooth](#3-emparejar-micrófono-bluetooth)  
  4. [Instalar fast-whisper.cpp](#4-instalar-fast-whispercpp)  
- [Uso](#uso)  
  - [Iniciar servicios de red](#iniciar-servicios-de-red)  
  - [Ejecutar el servidor de transcripción](#ejecutar-el-servidor-de-transcripción)  
  - [Acceder a la interfaz web](#acceder-a-la-interfaz-web)  
- [Optimización y troubleshooting](#optimización-y-troubleshooting)  
- [Contribuciones](#contribuciones)  
- [Licencia](#licencia)  

---

## Descripción

**Inclu-IA** es un sistema de subtitulado en tiempo real para aulas, basado en Raspberry Pi 4 y un micrófono Bluetooth, que captura la voz del profesor, la transcribe usando IA (fast-whisper.cpp) y la distribuye vía un punto de acceso Wi-Fi a una página web de subtítulos para estudiantes con discapacidad auditiva.

---

## Características

- Captura de audio inalámbrica (Bluetooth)  
- Transcripción en streaming con **fast-whisper.cpp**  
- Punto de acceso Wi-Fi integrado (modo AP)  
- Cliente web ligero con subtítulos en tiempo real  
- Escalable: cambia de modelo **tiny**, **small**, **medium**, etc.  

---

## Requisitos

### Hardware

- **Raspberry Pi 4** con Raspbian (64-bit)  
- **Micrófono Bluetooth** (p. ej. BoomArm BT-800)  
- **Tarjeta SD** ≥ 16 GB (clase 10)  
- Fuente de alimentación estable  

### Software

- **SO**: Raspbian actualizado  
- **Paquetes básicos**:  
  - `git`, `build-essential`, `python3-pip`  
  - `hostapd`, `dnsmasq`, `avahi-daemon`  
- **Audio/Bluetooth**:  
  - `pulseaudio`, `pulseaudio-module-bluetooth`, `bluez-tools`  
  - `python3-pyaudio`  
- **Servidor y streaming**:  
  - Python 3: `Flask`, `Flask-SocketIO`, `python-socketio`, `PyAudio`  

---

## Estructura del repositorio

```bash
Inclu-IA/
├── README.md
├── hardware/
│   └── esquemas_diagrama.md
├── software/
│   ├── server.py
│   ├── requirements.txt
│   └── config/
│       ├── hostapd.conf
│       └── dnsmasq.conf
├── web/
│   └── templates/
│       └── index.html
└── docs/
    ├── instalación.md
    ├── uso.md
    └── troubleshooting.md
```

---

## Instalación

### 1. Actualizar Raspbian

```bash
sudo apt update && sudo apt upgrade -y
sudo reboot
```

Instala herramientas básicas:

```bash
sudo apt install -y git build-essential python3-pip \
  hostapd dnsmasq avahi-daemon
```

---

### 2. Configurar punto de acceso Wi-Fi

1. **Editar** `/etc/hostapd/hostapd.conf`:

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

2. **Apuntar** en `/etc/default/hostapd`:

   ```bash
   DAEMON_CONF="/etc/hostapd/hostapd.conf"
   ```

3. **Configurar DHCP** en `/etc/dnsmasq.conf`:

   ```ini
   interface=wlan0
   dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
   ```

4. **Asignar IP estática** en `/etc/dhcpcd.conf`:

   ```ini
   interface wlan0
     static ip_address=192.168.4.1/24
     nohook wpa_supplicant
   ```

5. **Habilitar y arrancar**:

   ```bash
   sudo systemctl enable hostapd dnsmasq
   sudo systemctl start hostapd dnsmasq
   ```

---

### 3. Emparejar micrófono Bluetooth

```bash
sudo apt install -y pulseaudio pulseaudio-module-bluetooth \
                    bluez-tools python3-pyaudio
```

Editar `/etc/pulse/system.pa` y añadir:

```ini
load-module module-bluetooth-policy
load-module module-bluetooth-discover
```

Luego:

```bash
bluetoothctl
# Dentro de bluetoothctl:
scan on
pair XX:XX:XX:XX:XX:XX
trust XX:XX:XX:XX:XX:XX
connect XX:XX:XX:XX:XX:XX
exit
```

Verifica que el micrófono esté disponible:

```bash
pactl list sources short
```

---

### 4. Instalar fast-whisper.cpp

1. **Clonar y compilar**:

   ```bash
   git clone https://github.com/ggerganov/whisper.cpp
   cd whisper.cpp
   make
   ```

2. **Descargar modelo** (e.g. `tiny`):

   ```bash
   ./models/download-ggml-model.sh tiny.en
   ```

3. **Probar con un archivo de audio**:

   ```bash
   arecord -f S16_LE -r 16000 -d 5 test.wav
   ./main -m models/ggml-tiny.en.bin -f test.wav
   ```

4. **Integración en el servidor**  
   Asegúrate de que `server.py` invoque `./main --stream` para procesar audio en tiempo real.

---

## Uso

### Iniciar servicios de red

```bash
sudo systemctl start hostapd dnsmasq
```

### Ejecutar el servidor de transcripción

```bash
cd software
pip3 install -r requirements.txt
python3 server.py
```

### Acceder a la interfaz web

1. Conéctate a la red **Subtitulos_AP** (clave: `TuClaveSegura`).  
2. Abre tu navegador en `http://192.168.4.1:5000`.  
   Verás los subtítulos en tiempo real.

---

## Optimización y troubleshooting

- **Modelos**: alterna entre `tiny`, `small`, `medium`, `large` según CPU y latencia.  
- **Memoria**: si falla la compilación, habilita swap adicional en Raspberry Pi.  
- **Ganancia**: ajusta la sensibilidad del micrófono en PulseAudio.  
- **Buffer**: en `server.py`, modifica `CHUNK` (pyaudio) para reducir latencia.  
- **Logs**: ejecuta `python3 server.py --debug` y añade `-v` a `./main` para más detalle.  

---

## Contribuciones

1. Haz un **fork** del repositorio.  
2. Crea una rama (`git checkout -b feature/mi-mejora`).  
3. Realiza **commits** claros y descriptivos.  
4. Abre un **pull request** detallando tus cambios.

---

## Licencia

Este proyecto está bajo licencia **MIT**. Consulta [LICENSE](LICENSE) para más información.
```
