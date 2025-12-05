# 🚀 PROYECTO FINAL SIS313: DISEÑO Y IMPLEMENTACIÓN DE UNA VPN SEGURA PARA ACCESO REMOTO (WIREGUARD)

> **Asignatura:** SIS313 – Infraestructura, Plataformas Tecnológicas y Redes  
> **Semestre:** 2/2025  
> **Docente:** Ing. Marcelo Quispe Ortega  

Infraestructura distribuida en **3 máquinas virtuales** con acceso seguro a través de un servidor **Proxy/VPN**, permitiendo la conexión de múltiples dispositivos móviles externos (vía datos móviles o WiFi).

---

## 👥 Miembros del Equipo

| Nombre Completo          | Rol en el Proyecto                   | Contacto (GitHub/Email) |
| :---                     | :---                                 | :--- |
| **Tito Gonzales Edgar**  | Administrador de Sistemas y Seguridad | @EDGAR320 |
| **Lenis Diaz Nair**      | Ingeniera de Redes y Automatización  | @Nair-Lenis |

---

## 🎯 1. Objetivo del Proyecto

> **Objetivo:**  
Diseñar e implementar una infraestructura segura de acceso remoto mediante **WireGuard**, distribuida en **3 VMs**, con un servidor Proxy/VPN encargado de gestionar las conexiones entrantes desde varios dispositivos externos (móviles u otros equipos), y dos VMs cliente (**Maestro** y **Estudiante**) para validar la creación del túnel VPN, el aislamiento de red y la transmisión cifrada extremo a extremo. :contentReference[oaicite:0]{index=0}  

---

## 💡 2. Justificación e Importancia

> **Justificación:**  
En un entorno académico o corporativo, el acceso remoto seguro es esencial para la continuidad operativa. Este proyecto implementa una **VPN moderna con WireGuard**, proporcionando cifrado robusto (T5), baja latencia y simplicidad de configuración.  
El uso de adaptadores puente en todas las VMs permite simular una red realista, mientras el servidor Proxy/VPN actúa como puerta de acceso controlado desde Internet.  
La funcionalidad de múltiples móviles conectados garantiza la escalabilidad del servicio y refleja escenarios reales de trabajo remoto.

---

## 🛠️ 3. Tecnologías y Conceptos Implementados

### 3.1. Tecnologías Clave

- **WireGuard:** Protocolo VPN moderno con cifrado avanzado y alto rendimiento.  
- **Docker & Docker-Compose:** Contenedorización y despliegue del servidor VPN.  
- **UFW / iptables:** Reglas de seguridad, forwarding y control del tráfico.  
- **Imagen LinuxServer.io (lscr.io/linuxserver/wireguard):** Gestión simplificada de peers y configuraciones. :contentReference[oaicite:1]{index=1}  
- **Ubuntu Server 22.04:** Sistema operativo base para las VMs.  
- **Cliente WireGuard Mobile:** Aplicación para conexión de **varios móviles** mediante códigos QR.  
- **SSH:** Administración segura de las máquinas virtuales.

### 3.2. Conceptos de la Asignatura (T1 – T6) Aplicados

- ✅ **Seguridad en Redes (T3, T5):** Túneles VPN cifrados, aislamiento de red, uso de claves públicas/privadas.  
- ✅ **Infraestructura y Servicios (T1):** Despliegue de servicios VPN sobre VMs Linux.  
- ✅ **Automatización (T6):** Uso de Docker-Compose para orquestar el servidor WireGuard.  
- ✅ **Gobernanza y Control (T4):** Verificación de peers, control de acceso y monitoreo básico mediante comandos `wg`.

---

## 🌐 4. Diseño de la Infraestructura y Topología

### 4.1. Componentes y Roles (3 VMs + múltiples móviles)

| VM / Host      | Hostname     | Rol                                  | Adaptador de Red | IP Asignada (LAN) | Sistema Operativo |
| :---           | :---         | :---                                 | :---             | :---              | :--- |
| **VM 1**       | proxyserver  | Servidor VPN / Proxy / Generador de Peers | Adaptador Puente | `192.168.100.64`  | Ubuntu 22.04 |
| **VM 2**       | maestro      | Cliente VPN (Peer2)                  | Adaptador Puente | `192.168.100.65`  | Ubuntu 22.04 |
| **VM 3**       | estudiante   | Cliente VPN (Peer3)                  | Adaptador Puente | `192.168.100.66`  | Ubuntu 22.04 |
| **Móviles**    | –            | Conexion a la misma red        | LTE/4G/5G        | IP dinámica móvil | Android / iOS |

> **Nota:** Todas las VMs están en la misma LAN física (adaptador puente) para simular un entorno de campus, mientras los móviles se conectan desde Internet a través de la IP pública del servidor VPN.

---

### 4.2. Diagrama Lógico de Arquitectura (Descripción)

- Los **móviles** se conectan por Internet a la IP pública del servidor (redirección al puerto UDP 51820).  
- El **servidor proxyserver** levanta WireGuard en Docker y genera los peers (archivos `.conf` y códigos QR).  
- Las VMs **maestro** y **estudiante** se conectan como clientes a la VPN, utilizando archivos de configuración (`wg0.conf`).  
- Una vez establecido el túnel, todos los peers comparten una red privada virtual (subred interna configurada en WireGuard, p.ej. `10.13.13.0/24`).

---

## 📋 5. Guía de Implementación y Puesta en Marcha

### 5.1. Pre-requisitos
Antes de iniciar la instalación:

- 3 máquinas virtuales con **Ubuntu Server 22.04 LTS**.  
- Todas las VMs en **adaptador puente** (mismo segmento LAN).  
- Acceso con usuario con permisos `sudo`.  
- Enrutamiento desde Internet al puerto **UDP 51820** de la IP pública del servidor.  
- Un dispositivo móvil (Android/iOS) con la app **WireGuard** instalada.

---

### 5.2. Despliegue Paso a Paso
La guía se divide en:

1. Configuración del **Servidor VPN (proxyserver)**  
2. Conexión de **dispositivos móviles**  
3. Configuración del **Cliente Estudiante (VM estudiante)**  
4. Configuración del **Cliente Maestro (VM maestro)**  

---

##🔹 6) Despliegue del Servidor VPN `proxyserver` (WireGuard + Docker)
En esta sección se detalla el **despliegue completo del servidor VPN** sobre la máquina virtual `proxyserver`, utilizando Docker y la imagen `lscr.io/linuxserver/wireguard`.  
El objetivo es dejar un servicio **seguro, persistente y administrable** desde un panel web.

---
### 6.1. Verificar la Red y la IP Pública

```bash
curl ipinfo.io
```
### 6.1. Preparar el SO e Instalar Docker

En la VM `proxyserver`:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose iptables-persistent qrencode
sudo usermod -aG docker $USER
```
Tras esto, cierra sesión y vuelve a entrar para que el grupo docker se aplique al usuario.
### 6.2. Crear la Estructura y docker-compose.yml
```
sudo mkdir -p /opt/wireguard-server
sudo chown $USER:$USER /opt/wireguard-server
cd /opt/wireguard-server
```

Crear el archivo /opt/wireguard-server/docker-compose.yml con:
```
version: "3.8"
services:
  wireguard:
    image: lscr.io/linuxserver/wireguard:latest
    container_name: wireguard
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Etc/UTC
      - SERVERURL=TU_IP_PUBLICA_O_DOMINIO
      - SERVERPORT=51820
      - PEERS=4              # ajusta: peers esperados (maestro, estudiante, movil1, movil2..)
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.13.13.0
      - ALLOWEDIPS=0.0.0.0/0
      - LOG_CONFS=true
    volumes:
      - /opt/wireguard-server/config:/config
      - /lib/modules:/lib/modules
    ports:
      - 51820:51820/udp
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
    restart: unless-stopped
```
#### 6.3. Levantar el Contenedor y Verificar
```
cd /opt/wireguard-server
docker-compose up -d
docker ps | grep wireguard
```

Ver logs y archivos generados:
```
docker logs wireguard --tail 200
ls -l /opt/wireguard-server/config
```
## 🌐7) NAT, Forwarding y Persistencia
Para que los clientes VPN puedan navegar a Internet a través del servidor, se configura reenvío IPv4 y NAT.

### 7.1. Habilitar el Reenvío IPv4
```
echo "net.ipv4.ip_forward=1" | sudo tee /etc/sysctl.d/99-wireguard.conf
sudo sysctl -p /etc/sysctl.d/99-wireguard.conf
```
### 7.2. Reglas iptables (ajustar ens33 a tu interfaz externa)
```
sudo iptables -t nat -A POSTROUTING -s 10.13.13.0/24 -o ens33 -j MASQUERADE
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT
```
### 7.3. Guardar Reglas para Reinicios
```
sudo apt install -y iptables-persistent
sudo netfilter-persistent save
```
## 🖥️ 8) Panel Web de Administración — Archivos y Permisos
El panel web permite consultar el estado del túnel, levantar/bajar WireGuard y gestionar archivos (subidas/descargas) de forma controlada.

Ruta base:
```
/var/www/html/vpn-panel
```
### 8.1. Crear Rutas y Ajustar Permisos
```
sudo mkdir -p /var/www/html/vpn-panel/data
sudo chown -R root:www-data /var/www/html/vpn-panel
sudo chmod -R 750 /var/www/html/vpn-panel
sudo chown www-data:www-data /var/www/html/vpn-panel/data
sudo chmod 770 /var/www/html/vpn-panel/data
```
### 8.2. config.php (Conexión SQLite)
```
<?php
$db_path = __DIR__ . '/data/db.sqlite';
try {
    $db = new PDO('sqlite:' . $db_path);
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    $db->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_ASSOC);
} catch (Exception $e) {
    die("Error al conectar con la base de datos: " . $e->getMessage());
}
?>

8.3. auth.php (Sesiones y Timeout)
<?php
session_start();
$timeout = 1800;

if (!isset($_SESSION['user'])) {
    header("Location: login.php");
    exit;
}

if (isset($_SESSION['LAST_ACTIVITY']) && (time() - $_SESSION['LAST_ACTIVITY']) > $timeout) {
    session_unset();
    session_destroy();
    header("Location: login.php?timeout=1");
    exit;
}

$_SESSION['LAST_ACTIVITY'] = time();

function require_role($role) {
    if (!isset($_SESSION['role']) || $_SESSION['role'] !== $role) {
        http_response_code(403);
        echo "ACCESO DENEGADO";
        exit;
    }
}
?>
```
### 8.4. login.php (Formulario y Verificación)
El archivo login.php se encarga de:

* Mostrar el formulario de acceso.

* Validar usuario/contraseña contra la tabla users.

* Crear las variables de sesión:
```
$_SESSION['user_id'], $_SESSION['user'], $_SESSION['role'].
```
(Puedes reutilizar el login.php que ya tienes implementado, ajustando únicamente rutas y nombres de tabla si es necesario.)

### 8.5. index.php — Panel Principal (Estado, Subida/Descarga)
Este archivo muestra el panel de control para el usuario autenticado:
```
<?php
require_once __DIR__.'/auth.php';
require_once __DIR__.'/config.php';
?>
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>Panel VPN</title>
</head>
<body>
  <h2>Panel VPN - <?=htmlspecialchars($_SESSION['user'])?></h2>

  <div>
    <button onclick="runAction('status')">Estado</button>
    <button onclick="runAction('up')">Levantar túnel</button>
    <button onclick="runAction('down')">Bajar túnel</button>
    <pre id="wgout">-- Esperando --</pre>
  </div>

  <div>
    <form id="formUp" enctype="multipart/form-data">
      <input type="file" name="file" required>
      <button>Subir</button>
    </form>
    <div id="files"></div>
  </div>

<script>
function runAction(action){
  const f = new FormData();
  f.append('action', action);
  fetch('ajax.php', {method:'POST', body:f})
    .then(r => r.json())
    .then(j => {
      document.getElementById('wgout').textContent = j.out || JSON.stringify(j);
    });
}

document.getElementById('formUp').addEventListener('submit', e => {
  e.preventDefault();
  const f = new FormData(e.target);
  fetch('upload.php', {method:'POST', body:f})
    .then(r => r.json())
    .then(j => {
      alert(j.msg);
      loadFiles();
    });
});

function loadFiles(){
  fetch('files_list.php')
    .then(r => r.text())
    .then(t => {
      document.getElementById('files').innerHTML = t;
    });
}

loadFiles();
</script>
</body>
</html>
```
### 8.6. ajax.php (Ejecución de Scripts Controlados)
```
<?php
require_once __DIR__.'/auth.php';
require_once __DIR__.'/config.php';

if ($_SERVER['REQUEST_METHOD'] !== 'POST') { 
    http_response_code(405); 
    exit; 
}

$action = $_POST['action'] ?? '';

if ($action === 'status') {
    exec('sudo /opt/wireguard-server/scripts/status.sh 2>&1', $out, $ret);
    echo json_encode(['code'=>$ret,'out'=>implode("\n",$out)]);
    exit;
}

if ($action === 'up') {
    exec('sudo /opt/wireguard-server/scripts/up.sh 2>&1', $out, $ret);
    echo json_encode(['code'=>$ret,'out'=>implode("\n",$out)]);
    exit;
}

if ($action === 'down') {
    exec('sudo /opt/wireguard-server/scripts/down.sh 2>&1', $out, $ret);
    echo json_encode(['code'=>$ret,'out'=>implode("\n",$out)]);
    exit;
}

http_response_code(400);
echo json_encode(['error'=>'acción desconocida']);
?>
```
### 8.7. upload.php, files_list.php, download.php (Gestión de Archivos)
Resumen funcional:

* upload.php:

    * Guarda en /var/www/uploads/<usuario>/
    * Registra en la tabla uploads
    * Calcula SHA256 del archivo

* files_list.php:

    * Lista archivos del usuario autenticado

* download.php:

    * Fuerza descarga
    * Registra la acción en la base

Asegura permisos coherentes:
```
sudo chown -R root:www-data /var/www/html/vpn-panel
sudo find /var/www/html/vpn-panel -type f -exec chmod 640 {} \;
sudo find /var/www/html/vpn-panel -type d -exec chmod 750 {} \;
sudo chown www-data:www-data /var/www/html/vpn-panel/data/db.sqlite
sudo chmod 660 /var/www/html/vpn-panel/data/db.sqlite
```

Configura Nginx para servir /var/www/html (sitio por defecto o vpn-panel) y habilita PHP-FPM (socket o puerto, según tu instalación).

## 🧩 9) Scripts del Servidor y sudoers
Estos scripts permiten que el panel web controle WireGuard de forma segura y limitada.

### 9.1. Crear Scripts en /opt/wireguard-server/scripts/
status.sh
```
#!/usr/bin/env bash
if docker ps --format '{{.Names}}' | grep -q wireguard; then
  docker exec wireguard /app/show-peers --json 2>/dev/null || docker exec wireguard wg show all 2>/dev/null
else
  wg show all 2>/dev/null || echo "WireGuard no parece estar corriendo"
fi
```
up.sh
```
#!/usr/bin/env bash
cd /opt/wireguard-server || { echo "No existe /opt/wireguard-server"; exit 1; }
docker-compose up -d 2>&1 || (docker compose up -d 2>&1)
```
down.sh
```
#!/usr/bin/env bash
cd /opt/wireguard-server || { echo "No existe /opt/wireguard-server"; exit 1; }
docker-compose down 2>&1 || (docker compose down 2>&1)
```

Permisos:
```
sudo mkdir -p /opt/wireguard-server/scripts
sudo chown root:root /opt/wireguard-server/scripts/*.sh
sudo chmod 750 /opt/wireguard-server/scripts/*.sh
```
### 9.2. sudoers para www-data

Archivo /etc/sudoers.d/www-data-wireguard:
```
www-data ALL=(root) NOPASSWD: /opt/wireguard-server/scripts/status.sh, /opt/wireguard-server/scripts/up.sh, /opt/wireguard-server/scripts/down.sh
```

Aplicar permisos:
```
sudo chmod 440 /etc/sudoers.d/www-data-wireguard
```
Con esto, el usuario del servidor web (www-data) puede ejecutar solo esos scripts específicos, sin contraseña.

## 🖧 10) Conectar una VM Cliente (estudiante / maestro)
Ejemplo usando el peer2 (ajustable):

En proxyserver:
```
ls -l /opt/wireguard-server/config/peer2/peer2.conf
cat /opt/wireguard-server/config/peer2/peer2.conf
```

Copiar al cliente:
```
scp usuario@192.168.100.64:/opt/wireguard-server/config/peer2/peer2.conf ./
```

En la VM cliente:
```
sudo mv peer2.conf /etc/wireguard/wg0.conf
sudo chmod 600 /etc/wireguard/wg0.conf
sudo wg-quick up wg0
ip a show wg0       # ver IP asignada ej: 10.13.13.2
sudo wg show
```

Pruebas:
```
ping -c 4 10.13.13.1      # ping al servidor VPN
curl http://10.13.13.1    # portal interno
curl google.com           # si ALLOWEDIPS=0.0.0.0/0 y NAT está OK
```

Apagar interfaz:
```
sudo wg-quick down wg0
```

## 📱 11) Cliente Móvil (Conexión por QR)

En el contenedor WireGuard, generar QR:
```
docker exec -it wireguard sh -c "qrencode -t ansiutf8 < /config/peer1/peer1.conf"
# o, si la imagen lo soporta:
docker exec -it wireguard /app/show-peer 1
```

En la app móvil WireGuard:

* Añadir túnel
* Escanear código QR
* Activar túnel

Verificar en el servidor:
```
docker exec -it wireguard wg
```
## ✅12) Pruebas y Verificación (Checklist)

* docker ps muestra el contenedor wireguard.

* docker exec -it wireguard wg lista peers y último handshake.

* Desde las VMs cliente:

    * ping 10.13.13.1 responde correctamente.
    * curl http://10.13.13.1 devuelve el portal web interno.
    * curl google.com funciona si NAT y ALLOWEDIPS están bien configurados.

* Panel web:

    * Login funciona.
    * Botón Estado muestra salida de status.sh.
    * Subida/descarga de archivos funciona (/var/www/uploads/<user>/).

* Logs:

    * docker logs wireguard
    * /var/log/nginx/error.log
log

## 🛡️ 13) Buenas Prácticas y Backups

* No subir /opt/wireguard-server/config/* a repos públicos.
  
* Mantener claves y archivos .conf fuera del control de versiones.

* Backups periódicos de:

    * /opt/wireguard-server/config
    * /var/www/html/vpn-panel/data/db.sqlite

* Permisos recomendados:

    * Archivos PHP: 640 root:www-data
    * Directorios: 750
    * DB SQLite: 660 www-data:www-data
    * Carpeta uploads: 2770 (setgid para heredar grupo).

* Revisar y rotar logs:

    * docker logs
    * logs de Nginx / PHP.

* Firewall:

    * Restringir SSH.
    * Exponer solo puertos necesarios (51820/udp, 80/443 si aplica).

* Actualizaciones:

    * docker pull lscr.io/linuxserver/wireguard
    * Reiniciar contenedor de forma controlada.
      
## 🧾 14) Conclusión (para la Entrega)
Se implementó una VPN segura y funcional con WireGuard sobre Docker, integrando máquinas virtuales y clientes móviles, con NAT para salida a Internet y un panel web para gestión y trazabilidad de acciones.
La solución es reproducible, escalable y cumple los requerimientos de seguridad y operatividad establecidos en la asignatura SIS313.

## 📎 15) Anexos (para el Repositorio)

* /opt/wireguard-server/docker-compose.yml (ejemplo de despliegue).

* /opt/wireguard-server/scripts/{status.sh,up.sh,down.sh}.

* /var/www/html/vpn-panel/{config.php,auth.php,login.php,index.php,ajax.php,upload.php,files_list.php,download.php}.

* docs/guia_instalacion.md (este documento).

* docs/schema_sqlite.sql (DDL para crear tablas: users, peers, uploads, actions, connections).


