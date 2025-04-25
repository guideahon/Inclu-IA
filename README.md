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

SubtItulIA es un sistema de subtitulado en tiempo real para aulas, basado en Raspberry Pi 4 y un micrófono Bluetooth, que captura la voz del profesor, la transcribe usando IA (fast-whisper.cpp) y la emite vía un punto de acceso Wi-Fi a una página web de subtítulos para estudiantes con discapacidad auditiva.

---

## Nombres propuestos

1. SubtItulIA  
2. AulaInClu-IA  
3. Transcr-IA-Pi  
4. CaptaIA Voz  
5. InstaSubtITul-IA  
6. Prof-IA-Captura  
7. ClassIA-Caption  
8. SubtítuloDinám-IA  
9. InClu-IA-live  
10. SubtItuladorIA  

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
