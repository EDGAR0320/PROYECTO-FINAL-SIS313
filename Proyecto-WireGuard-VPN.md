## PROYECTO FINAL SIS313: DISEÑO Y IMPLEMENTACION DE UNA VPN SEGURA PARA ACCESO REMOTO (WIREGUARD)

> **Asignatura:** SIS313 – Infraestructura, Plataformas Tecnológicas y Redes  
> **Semestre:** 2/2025  
> **Docente:** Ing. Marcelo Quispe Ortega  

Infraestructura distribuida en **3 máquinas virtuales** con acceso seguro a través de un servidor **Proxy/VPN**, permitiendo la conexión de múltiples dispositivos móviles externos.

---

## 👥 Miembros del Equipo

| Nombre Completo | Rol en el Proyecto | Contacto (GitHub/Email) |
| :--- | :--- | :--- |
| **Tito Gonzales Edgar** | Administrador de Sistemas y Seguridad | @EDGAR320 |
| **Lenis Diaz Nair** | Ingeniera de Redes y Automatización | @Nair-Lenis |

---

## 🎯 I. Objetivo del Proyecto

> **Objetivo:**  
Diseñar e implementar una infraestructura segura de acceso remoto mediante **WireGuard**, distribuida en **3 VMs**, con un servidor Proxy/VPN encargado de gestionar las conexiones entrantes desde varios dispositivos externos (móviles u otros equipos), y dos VMs cliente (Maestro y Estudiante) para validar la creación del túnel VPN, el aislamiento de red y la transmisión cifrada extremo a extremo.

---

## 💡 II. Justificación e Importancia

> **Justificación:**  
En un entorno académico o corporativo, el acceso remoto seguro es esencial para la continuidad operativa. Este proyecto implementa una **VPN moderna con WireGuard**, proporcionando cifrado robusto (T5), baja latencia y simplicidad de configuración.  
El uso de adaptadores puente en todas las VMs permite simular una red realista, mientras el servidor Proxy/VPN actúa como puerta de acceso controlado desde Internet.  
La funcionalidad de múltiples móviles conectados garantiza la escalabilidad del servicio y refleja escenarios reales de trabajo remoto.

---

## 🛠️ III. Tecnologías y Conceptos Implementados

### 3.1. Tecnologías Clave

- **WireGuard:** Protocolo VPN moderno con cifrado avanzado y alto rendimiento.  
- **Docker & Docker-Compose:** Contenedorización y despliegue del servidor VPN.  
- **UFW / iptables:** Reglas de seguridad, forwarding y control del tráfico.  
- **Imagen LinuxServer.io:** Gestión simplificada de peers y configuraciones.  
- **Ubuntu Server 22.04:** Sistema operativo base para las VMs.  
- **Cliente WireGuard Mobile:** Aplicación para conexión de **varios móviles** mediante códigos QR.  
- **SSH:** Administración segura de las máquinas virtuales.

---

## 🌐 IV. Diseño de la Infraestructura y Topología

### 4.1. Componentes y Roles (3 VMs + múltiples móviles)

| VM / Host | Hostname | Rol | Adaptador de Red | IP Asignada | Sistema Operativo |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **VM 1** | proxyserver | Servidor VPN / Proxy / Generador de Peers | Adaptador Puente | 192.168.100.64 | Ubuntu 22.04 |
| **VM 2** | maestro | Cliente VPN (Peer2) | Adaptador Puente | 192.168.100.120 | Ubuntu 22.04 |
| **VM 3** | estudiante | Cliente VPN (Peer3) | Adaptador Puente | 192.168.100.150 | Ubuntu 22.04 |
| **Móviles externos** | – | Peers 1, 2, 3, ... conectados por datos móviles | LTE/4G/5G | IP Pública Operadora | Android / iOS |

---

### 4.2. Diagrama de Arquitectura

#### Tabla de Direcciones IP

| VM/Host | Hostname | Rol | IP Local (LAN) | Red | SO |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **ProxyServer** | proxyserver | Servidor VPN WireGuard + Proxy | 192.168.100.64 | LAN | Ubuntu 22.04 |
| **VM Maestro** | maestro | Cliente VPN Peer2 | 192.168.100.65 | LAN | Ubuntu 22.04 |
| **VM Estudiante** | estudiante | Cliente VPN Peer3 | 192.168.100.66 | LAN | Ubuntu 22.04 |
| **Móviles** | – | Peers móviles remotos (1..N) | IP dinámica móvil | Internet | Android / iOS |

#### Diagrama de Arquitectura


