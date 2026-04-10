# 🦠 Práctica 8 — Implementación y Configuración de ClamAV en Linux

> **Materia:** Seguridad de Sistemas Operativos  
> **Instructor:** Juan Alexander Ramírez  
> **Autores:** Neury Montero Mejia & Miguel Ramirez Meli
> **Institución:** Instituto Tecnológico de Las Américas (ITLA)  
> **Fecha:** 29-3-2026

---

## 📋 Tabla de Contenidos

### 🔴 Versión RHEL
- [Punto A — Implementar ClamAV (RHEL)](#-punto-a--implementar-clamav-rhel)
- [Punto B — Configurar actualizaciones automáticas (RHEL)](#-punto-b--configurar-actualizaciones-automáticas-rhel)
- [Punto C — Configurar escaneos periódicos (RHEL)](#-punto-c--configurar-escaneos-periódicos-rhel)
- [Punto D — Mostrar logs generados por ClamAV (RHEL)](#-punto-d--mostrar-logs-generados-por-clamav-rhel)

### 🐧 Versión Debian
- [Punto A — Implementar ClamAV (Debian)](#-punto-a--implementar-clamav-debian)
- [Punto B — Configurar actualizaciones automáticas (Debian)](#-punto-b--configurar-actualizaciones-automáticas-debian)
- [Punto C — Configurar escaneos periódicos (Debian)](#-punto-c--configurar-escaneos-periódicos-debian)
- [Punto D — Mostrar logs generados por ClamAV (Debian)](#-punto-d--mostrar-logs-generados-por-clamav-debian)

### 📊 [Diferencias clave: RHEL vs Debian](#-diferencias-clave-rhel-vs-debian)

---

# 🔴 VERSIÓN RHEL

> Distribución utilizada: **Red Hat Enterprise Linux (RHEL 8)**

---

## 🛠️ Punto A — Implementar ClamAV (RHEL)

> **Objetivo:** Instalar y poner en funcionamiento ClamAV en RHEL.

### Paso 1 — Actualizar los repositorios del sistema

Estando como root (`su`), ejecuta:

```bash
[root@localhost Neury0829]# dnf update -y
```

Si no hay actualizaciones disponibles, verás:

```
Dependencies resolved.
Nothing to do.
Complete!
```

### Paso 2 — Activar el repositorio CodeReady Builder

Este repositorio contiene dependencias necesarias para ClamAV:

```bash
[root@localhost Neury0829]# sudo subscription-manager repos --enable codeready-builder-for-rhel-8-$(arch)-rpms
```

Resultado esperado:

```
Repository 'codeready-builder-for-rhel-8-x86_64-rpms' is enabled for this system.
```

> ⚠️ En algunas distros este repositorio ya viene activado por defecto.

### Paso 3 — Instalar el repositorio EPEL

EPEL (Extra Packages for Enterprise Linux) es donde se encuentra el paquete de ClamAV:

```bash
[root@localhost Neury0829]# sudo dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

> ⚠️ Es importante activar CodeReady Builder **antes** de instalar EPEL, ya que muchos paquetes de EPEL dependen de él.

### Paso 4 — Actualizar nuevamente los repositorios

```bash
[root@localhost Neury0829]# dnf update -y
```

### Paso 5 — Instalar ClamAV

```bash
[root@localhost Neury0829]# sudo dnf -y install clamav clamd clamav-update
```

### Paso 6 — Verificar la instalación

```bash
[root@localhost Neury0829]# clamscan --version
ClamAV 1.4.3
```

> ℹ️ ClamAV incluye dos herramientas principales:
> - **ClamScan**: escanea el sistema en busca de virus.
> - **FreshClam**: actualiza la base de datos de firmas de virus.

### Paso 7 — Actualizar la base de datos con FreshClam

```bash
[root@localhost Neury0829]# sudo freshclam
```

FreshClam descargará las bases de datos `daily.cvd`, `main.cvd` y `bytecode.cvd`.

### Paso 8 — Crear el archivo de log y establecer permisos

```bash
[root@localhost Neury0829]# touch /var/log/freshclam.log
[root@localhost Neury0829]# chmod 600 /var/log/freshclam.log
[root@localhost Neury0829]# chown root /var/log/freshclam.log
```

| Comando | Descripción |
|---|---|
| `touch` | Crea el archivo de log vacío |
| `chmod 600` | Solo root puede leer y escribir el archivo |
| `chown root` | Asigna la propiedad del archivo a root |

### Paso 9 — Iniciar el servicio FreshClam

```bash
[root@localhost Neury0829]# systemctl start clamav-freshclam.service
```

### Paso 10 — Verificar que el servicio está activo

```bash
[root@localhost Neury0829]# systemctl status clamav-freshclam.service
```

Resultado esperado:

```
● clamav-freshclam.service - ClamAV virus database updater
   Loaded: loaded (/usr/lib/systemd/system/clamav-freshclam.service)
   Active: active (running) since Wed 2026-03-04 15:21:35 EST
```

✅ **ClamAV instalado y el servicio FreshClam activo.**

---

## 🔄 Punto B — Configurar actualizaciones automáticas (RHEL)

> **Objetivo:** Configurar FreshClam para actualizar automáticamente la base de datos de virus.

### Paso 1 — Abrir la configuración global de FreshClam

```bash
[root@localhost Neury0829]# nano /etc/freshclam.conf
```

### Paso 2 — Descomentar `DatabaseMirror`

Busca la línea comentada y descoméntala:

```
DatabaseMirror database.clamav.net
```

> ℹ️ Esto activa el dominio primario de ClamAV (CDN global de CloudFlare) para descargar actualizaciones.

### Paso 3 — Descomentar `UpdateLogFile`

Busca y descomenta:

```
UpdateLogFile /var/log/freshclam.log
```

> ℹ️ Esto habilita el registro de logs de las actualizaciones de la base de datos.

### Paso 4 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

### Paso 5 — Habilitar el servicio para que inicie automáticamente

```bash
[root@localhost Neury0829]# systemctl enable clamav-freshclam
```

Resultado esperado:

```
Created symlink /etc/systemd/system/multi-user.target.wants/clamav-freshclam.service → /usr/lib/systemd/system/clamav-freshclam.service.
```

✅ **FreshClam se actualizará automáticamente al iniciar el sistema.**

---

## ⏰ Punto C — Configurar escaneos periódicos (RHEL)

> **Objetivo:** Crear un script de escaneo y programarlo para ejecutarse diariamente a las 3:00 AM con crontab.

### Paso 1 — Crear el script de escaneo

```bash
[root@localhost Neury0829]# nano /usr/local/bin/escaneo_clamav.sh
```

### Paso 2 — Escribir el contenido del script

Dentro del archivo, escribe lo siguiente:

```bash
#!/bin/bash
LOGFILE="/var/log/clamav_scan.log"
clamscan -r /home /var --move=/quarantine --log=$LOGFILE
```

| Parte del script | Descripción |
|---|---|
| `#!/bin/bash` | Indica que el script se ejecuta en Bash |
| `LOGFILE` | Ruta donde se guardarán los registros del escaneo |
| `clamscan -r` | Escaneo recursivo (entra en subdirectorios) |
| `/home /var` | Directorios a escanear |
| `--move=/quarantine` | Mueve los archivos infectados a cuarentena |
| `--log=$LOGFILE` | Guarda el resultado en el archivo de log |

### Paso 3 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

### Paso 4 — Dar permisos de ejecución al script

```bash
[root@localhost Neury0829]# chmod +x /usr/local/bin/escaneo_clamav.sh
```

### Paso 5 — Abrir el crontab para programar la tarea

```bash
[root@localhost Neury0829]# sudo crontab -e
```

### Paso 6 — Agregar la tarea programada

Presiona `I` para entrar en modo de inserción (editor vi/vim) y escribe:

```
0 3 * * * /usr/local/bin/escaneo_clamav.sh
```

| Campo | Valor | Significado |
|---|---|---|
| Minuto | `0` | Al minuto 0 |
| Hora | `3` | A las 3:00 AM |
| Día del mes | `*` | Todos los días |
| Mes | `*` | Todos los meses |
| Día de la semana | `*` | Cualquier día |

> ✅ El script se ejecutará **todos los días a las 3:00 AM**.

### Paso 7 — Guardar y salir del crontab

```
Esc → :wq → Enter
```

✅ **Escaneos periódicos configurados correctamente.**

---

## 📋 Punto D — Mostrar logs generados por ClamAV (RHEL)

> **Objetivo:** Ejecutar un escaneo manual y visualizar los resultados en los logs.

### Paso 1 — Ejecutar un escaneo manual del directorio home

```bash
[root@localhost Neury0829]# clamscan -r /home /var --log=/var/log/clamav_scan.log
```

### Paso 2 — Esperar a que finalice el escaneo

El escaneo puede tardar varios minutos dependiendo del tamaño del sistema.

### Paso 3 — Ver el resumen del escaneo (SCAN SUMMARY)

Al finalizar, verás un resumen similar a este:

```
----------- SCAN SUMMARY -----------
Known viruses:       3627601
Engine version:      1.4.3
Scanned directories: 994
Scanned files:       1673
Infected files:      0
Data scanned:        1717.32 MB
Data read:           1292.13 MB (ratio 1.33:1)
Time:                172.557 sec (2 m 52 s)
Start Date:          2026:03:04 15:26:41
End Date:            2026:03:04 15:29:34
```

### Paso 4 — Ver los logs guardados en el archivo

```bash
[root@localhost Neury0829]# cat /var/log/clamav_scan.log
```

✅ **Los logs muestran el detalle completo del escaneo, incluyendo archivos revisados y posibles amenazas.**

---
---

# 🐧 VERSIÓN DEBIAN

> Distribución utilizada: **Debian / Ubuntu**

---

## 🛠️ Punto A — Implementar ClamAV (Debian)

> **Objetivo:** Instalar y poner en funcionamiento ClamAV en Debian/Ubuntu.

### Paso 1 — Actualizar los repositorios del sistema

```bash
usuario@debian:~$ sudo apt update && sudo apt upgrade -y
```

> ℹ️ En Debian/Ubuntu no se necesita activar repositorios adicionales como EPEL. ClamAV está disponible directamente en los repositorios oficiales.

### Paso 2 — Instalar ClamAV

```bash
usuario@debian:~$ sudo apt install clamav clamav-daemon -y
```

| Paquete | Descripción |
|---|---|
| `clamav` | Incluye ClamScan y FreshClam |
| `clamav-daemon` | Servicio en segundo plano (equivalente a `clamd` en RHEL) |

### Paso 3 — Detener el servicio clamav-freshclam para actualizar la base de datos

```bash
usuario@debian:~$ sudo systemctl stop clamav-freshclam
```

### Paso 4 — Actualizar la base de datos manualmente

```bash
usuario@debian:~$ sudo freshclam
```

### Paso 5 — Verificar la instalación

```bash
usuario@debian:~$ clamscan --version
ClamAV 1.x.x
```

### Paso 6 — Crear el archivo de log y establecer permisos

```bash
usuario@debian:~$ sudo touch /var/log/freshclam.log
usuario@debian:~$ sudo chmod 600 /var/log/freshclam.log
usuario@debian:~$ sudo chown root /var/log/freshclam.log
```

### Paso 7 — Iniciar y verificar el servicio FreshClam

```bash
usuario@debian:~$ sudo systemctl start clamav-freshclam
usuario@debian:~$ sudo systemctl status clamav-freshclam
```

Resultado esperado:

```
● clamav-freshclam.service - ClamAV virus database updater
   Active: active (running)
```

✅ **ClamAV instalado y FreshClam activo en Debian.**

---

## 🔄 Punto B — Configurar actualizaciones automáticas (Debian)

> **Objetivo:** Configurar FreshClam para actualizaciones automáticas.

### Paso 1 — Abrir la configuración de FreshClam

```bash
usuario@debian:~$ sudo nano /etc/clamav/freshclam.conf
```

> ℹ️ En Debian, el archivo de configuración está en `/etc/clamav/freshclam.conf`, no en `/etc/freshclam.conf` como en RHEL.

### Paso 2 — Verificar que `DatabaseMirror` esté activo

Busca la línea:

```
DatabaseMirror database.clamav.net
```

Si está comentada, descoméntala.

### Paso 3 — Verificar que `UpdateLogFile` esté activo

Busca:

```
UpdateLogFile /var/log/clamav/freshclam.log
```

> ℹ️ En Debian la ruta del log es `/var/log/clamav/freshclam.log` en lugar de `/var/log/freshclam.log`.

### Paso 4 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

### Paso 5 — Habilitar el servicio para inicio automático

```bash
usuario@debian:~$ sudo systemctl enable clamav-freshclam
```

✅ **FreshClam se actualizará automáticamente en Debian.**

---

## ⏰ Punto C — Configurar escaneos periódicos (Debian)

> **Objetivo:** Crear un script de escaneo y programarlo con crontab.

### Paso 1 — Crear el directorio de cuarentena

```bash
usuario@debian:~$ sudo mkdir -p /quarantine
```

### Paso 2 — Crear el script de escaneo

```bash
usuario@debian:~$ sudo nano /usr/local/bin/escaneo_clamav.sh
```

### Paso 3 — Escribir el contenido del script

```bash
#!/bin/bash
LOGFILE="/var/log/clamav_scan.log"
clamscan -r /home /var --move=/quarantine --log=$LOGFILE
```

### Paso 4 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

### Paso 5 — Dar permisos de ejecución

```bash
usuario@debian:~$ sudo chmod +x /usr/local/bin/escaneo_clamav.sh
```

### Paso 6 — Abrir el crontab

```bash
usuario@debian:~$ sudo crontab -e
```

> ℹ️ En Debian, al abrir crontab por primera vez te pedirá elegir un editor. Se recomienda seleccionar `nano` (opción 1).

### Paso 7 — Agregar la tarea programada

Al final del archivo escribe:

```
0 3 * * * /usr/local/bin/escaneo_clamav.sh
```

### Paso 8 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

✅ **El escaneo se ejecutará todos los días a las 3:00 AM en Debian.**

---

## 📋 Punto D — Mostrar logs generados por ClamAV (Debian)

> **Objetivo:** Ejecutar un escaneo manual y visualizar los logs.

### Paso 1 — Ejecutar el escaneo manual

```bash
usuario@debian:~$ sudo clamscan -r /home /var --log=/var/log/clamav_scan.log
```

### Paso 2 — Esperar a que finalice

El escaneo puede tardar varios minutos.

### Paso 3 — Ver el resumen del escaneo

Al terminar verás el `SCAN SUMMARY` directamente en la terminal:

```
----------- SCAN SUMMARY -----------
Known viruses:       3627601
Engine version:      1.x.x
Scanned directories: 994
Scanned files:       1673
Infected files:      0
Data scanned:        1717.32 MB
Time:                172.557 sec (2 m 52 s)
```

### Paso 4 — Ver los logs desde el archivo

```bash
usuario@debian:~$ cat /var/log/clamav_scan.log
```

### Paso 5 (opcional) — Ver los logs de FreshClam en Debian

```bash
usuario@debian:~$ cat /var/log/clamav/freshclam.log
```

✅ **Los logs muestran el detalle completo del escaneo.**

---

## 📊 Diferencias clave: RHEL vs Debian

| Aspecto | RHEL / CentOS | Debian / Ubuntu |
|---|---|---|
| Gestor de paquetes | `dnf` | `apt` |
| Instalar ClamAV | `dnf install clamav clamd clamav-update` | `apt install clamav clamav-daemon` |
| Repositorio extra necesario | EPEL + CodeReady Builder | No necesario |
| Config FreshClam | `/etc/freshclam.conf` | `/etc/clamav/freshclam.conf` |
| Log FreshClam | `/var/log/freshclam.log` | `/var/log/clamav/freshclam.log` |
| Servicio SSH | `clamav-freshclam.service` | `clamav-freshclam` (igual) |
| Editor crontab | vi/vim (`:wq` para guardar) | nano (seleccionable al abrir) |
| Directorio config ClamAV | `/etc/` | `/etc/clamav/` |

---

## 📝 Resumen de Comandos

| Comando | Descripción |
|---|---|
| `dnf install clamav clamd clamav-update` | Instalar ClamAV en RHEL |
| `apt install clamav clamav-daemon` | Instalar ClamAV en Debian |
| `clamscan --version` | Verificar versión instalada |
| `freshclam` | Actualizar base de datos de virus |
| `systemctl start clamav-freshclam` | Iniciar el servicio FreshClam |
| `systemctl enable clamav-freshclam` | Habilitar FreshClam al inicio del sistema |
| `systemctl status clamav-freshclam` | Ver estado del servicio FreshClam |
| `clamscan -r /home /var --log=<ruta>` | Escanear directorios y guardar log |
| `cat /var/log/clamav_scan.log` | Ver logs del escaneo |
| `chmod +x <script>` | Dar permisos de ejecución a un script |
| `crontab -e` | Editar tareas programadas |
| `touch /var/log/freshclam.log` | Crear archivo de log |
| `chmod 600 <archivo>` | Permisos de lectura/escritura solo para root |
| `chown root <archivo>` | Asignar propiedad del archivo a root |

---

*Documentación realizada en RHEL (Red Hat Enterprise Linux) y Debian/Ubuntu*
