# 🔍 Segundo Parcial — Nessus: Escaneo de Vulnerabilidades

> **Materia:** Seguridad de Sistemas Operativos  
> **Instructor:** Juan Alexander Ramírez  
> **Autores:** Neury Montero Mejia & Miguel Ramirez Meli

---

## 📋 Tabla de Contenidos

### 🪟 Versión Windows Server (RHEL como VM)
- [¿Qué es Nessus?](#-qué-es-nessus)
- [Punto A — Implementar Nessus](#-punto-a--implementar-nessus)
- [Punto B — Crear vulnerabilidades en el servidor](#-punto-b--crear-vulnerabilidades-en-el-servidor)
- [Punto C — Configurar y ejecutar el escaneo](#-punto-c--configurar-y-ejecutar-el-escaneo)
- [Punto D y E — Ver resultados y aplicar soluciones CVE](#-punto-d-y-e--ver-resultados-y-aplicar-soluciones-cve)
- [Punto F — Nuevo reporte y conclusión](#-punto-f--nuevo-reporte-y-conclusión)

### 🐧 Versión Debian/Linux como objetivo de escaneo
- [Punto A — Implementar Nessus (Debian)](#-punto-a--implementar-nessus-debian)
- [Punto B — Crear vulnerabilidades (Debian)](#-punto-b--crear-vulnerabilidades-debian)
- [Punto C — Configurar y ejecutar el escaneo (Debian)](#-punto-c--configurar-y-ejecutar-el-escaneo-debian)
- [Punto D y E — Ver resultados y aplicar soluciones CVE (Debian)](#-punto-d-y-e--ver-resultados-y-aplicar-soluciones-cve-debian)
- [Punto F — Nuevo reporte y conclusión (Debian)](#-punto-f--nuevo-reporte-y-conclusión-debian)

### 📊 [Diferencias clave: Windows Server vs Debian](#-diferencias-clave-windows-server-vs-debian)

---

## 💡 ¿Qué es Nessus?

**Nessus** es una herramienta de escaneo de vulnerabilidades desarrollada por **Tenable**. Permite analizar sistemas, redes y servidores en busca de debilidades de seguridad conocidas, clasificándolas por severidad usando el estándar **CVE** (Common Vulnerabilities and Exposures).

Sus principales características son:

| Característica | Descripción |
|---|---|
| Escaneo de red | Detecta hosts activos, puertos abiertos y servicios expuestos |
| Detección de CVEs | Identifica vulnerabilidades conocidas con su ID y severidad |
| Clasificación CVSS | Califica las vulnerabilidades: Info, Low, Medium, High, Critical |
| Remediaciones | Proporciona soluciones concretas para cada vulnerabilidad encontrada |
| Plugins | Se actualiza constantemente con nuevas firmas de vulnerabilidades |

> ℹ️ La versión gratuita **Nessus Essentials** permite escanear hasta 16 IPs y es suficiente para entornos de laboratorio y práctica.

---

# 🪟 VERSIÓN WINDOWS SERVER

> Nessus instalado en **Windows**, escaneando un servidor **Windows Server 2022** con **Active Directory**.

---

## 🛠️ Punto A — Implementar Nessus

> **Objetivo:** Descargar, instalar y configurar Nessus Essentials en Windows.

### Paso 1 — Descargar Nessus

Ve al sitio oficial de Tenable y descarga el instalador para tu sistema operativo:

```
https://www.tenable.com/downloads/nessus
```

Selecciona:
- **Version:** Nessus (la más reciente, ej. 10.11.3)
- **Platform:** Windows - x86_64

El archivo descargado será algo como: `Nessus-10.11.3-x64.msi`

### Paso 2 — Instalar Nessus

Ejecuta el archivo `.msi` descargado y sigue el asistente de instalación.

### Paso 3 — Crear cuenta y obtener licencia

Al finalizar la instalación, Nessus abrirá automáticamente en el navegador en:

```
https://localhost:8834
```

- Selecciona **Nessus Essentials** (versión gratuita)
- Crea una cuenta en Tenable o inicia sesión
- Se generará automáticamente una clave de activación

### Paso 4 — Esperar la instalación de plugins

> ⚠️ **Importante:** Nessus debe descargar e instalar todos sus plugins antes de poder realizar escaneos. Este proceso puede tardar varios minutos. No podrás ejecutar escaneos hasta que finalice.

Una vez completado, la interfaz mostrará el botón **"New Scan"** habilitado.

✅ **Nessus instalado y listo para usarse.**

---

## ⚠️ Punto B — Crear vulnerabilidades en el servidor

> **Objetivo:** Introducir vulnerabilidades deliberadas en el servidor Windows para que Nessus las detecte (mínimo 3 de nivel Medium o superior).

### Paso 1 — Abrir Windows Defender

Busca `defender` en el buscador de Windows y selecciona **"Protección antivirus y contra amenazas"**.

### Paso 2 — Ir a "Administrar la configuración"

Dentro de la sección **"Configuración de antivirus y protección contra amenazas"**, haz clic en **"Administrar la configuración"**.

### Paso 3 — Desactivar las protecciones

Desactiva las siguientes opciones (cambia cada toggle de **Activado** a **Desactivado**):

| Protección | Estado resultante | Vulnerabilidad generada |
|---|---|---|
| Protección en tiempo real | ❌ Desactivado | Exposición a malware activo |
| Protección basada en la nube | ❌ Desactivado | Sin detección de amenazas en nube |
| Envío de muestras automático | ❌ Desactivado | Sin reporte de archivos sospechosos |
| Protección contra alteraciones | ❌ Desactivado | Permite modificar configuración de seguridad |

### Paso 4 — Desactivar el Firewall de Windows Defender

Ve a **"Firewall y protección de red"** y desactiva el firewall para:

- 🔴 **Red de dominio** — El firewall está desactivado
- 🔴 **Red privada** — El firewall está desactivado
- 🔴 **Red pública** — El firewall está desactivado

> ⚠️ Esto genera vulnerabilidades de nivel **High** y **Critical** que Nessus detectará en el escaneo.

✅ **Vulnerabilidades creadas en el servidor.**

---

## 🔎 Punto C — Configurar y ejecutar el escaneo

> **Objetivo:** Crear un nuevo escaneo en Nessus apuntando al servidor Windows con Active Directory.

### Paso 1 — Crear un nuevo escaneo

En la interfaz de Nessus (`https://localhost:8834`), haz clic en **"New Scan"**.

### Paso 2 — Seleccionar la plantilla

En **"Scan Templates"**, selecciona:

```
Basic Network Scan
```

> ℹ️ Esta plantilla realiza un escaneo completo de vulnerabilidades sin necesidad de configuración avanzada.

### Paso 3 — Obtener la IP del servidor objetivo

Abre el **Símbolo del sistema** (`cmd`) y ejecuta:

```
C:\Users\Administrador> ipconfig
```

Anota la **Dirección IPv4** (ej. `192.168.0.9`).

### Paso 4 — Configurar el escaneo

Rellena los campos del formulario:

| Campo | Valor |
|---|---|
| **Name** | Segundo Parcial |
| **Description** | Nessus |
| **Folder** | My Scans |
| **Targets** | 192.168.0.9 |

### Paso 5 — Configurar credenciales de Windows

Ve a la pestaña **"Credentials"** → **"Windows"** y completa:

| Campo | Valor |
|---|---|
| Authentication method | Password |
| Username | Administrador |
| Password | (contraseña del servidor) |
| Domain | Neury0829.local |

> ℹ️ Para verificar el dominio del servidor, ve al **Administrador del servidor** → **Servidor local** y busca el campo "Dominio".

### Paso 6 — Guardar y lanzar el escaneo

Haz clic en **"Save"** y luego en el botón **"Launch"** (▶) para iniciar el escaneo.

El escaneo puede tardar entre 15 y 30 minutos dependiendo del sistema.

✅ **Escaneo en ejecución.**

---

## 📊 Punto D y E — Ver resultados y aplicar soluciones CVE

> **Objetivo:** Analizar las vulnerabilidades detectadas y aplicar las soluciones recomendadas por Nessus.

### Paso 1 — Abrir el resultado del escaneo

Una vez completado el escaneo, haz clic en el nombre **"Segundo Parcial"** para ver los resultados.

### Paso 2 — Revisar el resumen de vulnerabilidades

En la vista del host `192.168.0.9` podrás ver un resumen como:

| Severidad | Cantidad (Escaneo 1) |
|---|---|
| 🔴 Critical | 21 |
| 🟠 High | 42 |
| 🟡 Medium | 8 |
| 🔵 Low | variable |
| ℹ️ Info | variable |

### Paso 3 — Examinar una vulnerabilidad específica

Haz clic sobre el host para ver el listado detallado. Al abrir una vulnerabilidad, verás:

- **ID del plugin** y versión
- **Descripción** del problema
- **CVE asociado** (ej. `CVE-2016-9535`, `CVE-2025-47827`)
- **Solución recomendada** (ej. "Apply Security Update 5066782")
- **Enlace de referencia** (ej. `https://support.microsoft.com/help/5066782`)

> ℹ️ El **CVE** (Common Vulnerabilities and Exposures) es un identificador estándar para vulnerabilidades conocidas. Cada CVE contiene la descripción completa del problema, el impacto y la solución oficial.

### Paso 4 — Aplicar las soluciones: Windows Update

La principal solución para vulnerabilidades de Windows es instalar las actualizaciones pendientes:

1. Ve a **Configuración** → **Actualización y seguridad**
2. En **Windows Update**, haz clic en **"Instalar ahora"**
3. Espera a que se descarguen e instalen todas las actualizaciones
4. Cuando aparezca "Es necesario reiniciar", haz clic en **"Reiniciar ahora"**

✅ **Actualizaciones aplicadas. Las vulnerabilidades relacionadas con parches quedarán resueltas.**

---

## 📋 Punto F — Nuevo reporte y conclusión

> **Objetivo:** Ejecutar un segundo escaneo para comparar resultados antes y después de aplicar las soluciones.

### Paso 1 — Crear un segundo escaneo

Repite el proceso del Punto C con el mismo target (`192.168.0.9`) pero nómbralo **"Segundo escaneo"**.

### Paso 2 — Comparar los resultados

| Severidad | Escaneo 1 (antes) | Escaneo 2 (después) | Reducción |
|---|---|---|---|
| 🔴 Critical | 21 | 0 | ✅ -100% |
| 🟠 High | 42 | 3 | ✅ -93% |
| 🟡 Medium | 8 | 4 | ✅ -50% |

### Conclusión

Nessus demostró ser una herramienta altamente efectiva para la detección y gestión de vulnerabilidades. Al aplicar las soluciones sugeridas (principalmente actualizaciones de Windows), se logró eliminar el **100% de las vulnerabilidades críticas** y reducir drásticamente las de nivel High y Medium. Esto valida la importancia de mantener los sistemas actualizados y de realizar escaneos periódicos de vulnerabilidades como parte de una estrategia de ciberseguridad proactiva.

---
---

# 🐧 VERSIÓN DEBIAN

> Nessus instalado en un sistema **Debian/Ubuntu**, escaneando una máquina virtual Linux como objetivo.

---

## 🛠️ Punto A — Implementar Nessus (Debian)

> **Objetivo:** Descargar e instalar Nessus Essentials en Debian/Ubuntu.

### Paso 1 — Descargar Nessus para Linux

Ve a la página de descargas de Tenable y selecciona el paquete para Debian:

```
https://www.tenable.com/downloads/nessus
```

Selecciona:
- **Version:** Nessus (la más reciente)
- **Platform:** Linux - Debian - amd64

O descárgalo directamente desde la terminal:

```bash
usuario@debian:~$ curl -o Nessus.deb "https://www.tenable.com/downloads/api/v2/pages/nessus/files/Nessus-<version>-debian10_amd64.deb"
```

### Paso 2 — Instalar el paquete

```bash
usuario@debian:~$ sudo dpkg -i Nessus-<version>-debian10_amd64.deb
```

### Paso 3 — Iniciar el servicio de Nessus

```bash
usuario@debian:~$ sudo systemctl start nessusd
usuario@debian:~$ sudo systemctl enable nessusd
```

### Paso 4 — Acceder a la interfaz web

Abre el navegador y ve a:

```
https://localhost:8834
```

> ⚠️ El navegador mostrará una advertencia de certificado. Acepta la excepción de seguridad para continuar.

### Paso 5 — Crear cuenta y activar licencia

- Selecciona **Nessus Essentials**
- Crea o inicia sesión con tu cuenta de Tenable
- Ingresa la clave de activación recibida por correo

### Paso 6 — Esperar instalación de plugins

> ⚠️ Al igual que en Windows, Nessus debe instalar todos sus plugins antes de poder escanear. Espera hasta que el botón **"New Scan"** se habilite.

✅ **Nessus instalado y listo en Debian.**

---

## ⚠️ Punto B — Crear vulnerabilidades (Debian)

> **Objetivo:** Introducir vulnerabilidades deliberadas en la VM Linux para que Nessus las detecte.

### Vulnerabilidad 1 — Deshabilitar el Firewall (ufw)

```bash
usuario@debian:~$ sudo ufw disable
```

### Vulnerabilidad 2 — Habilitar el acceso root por SSH

```bash
usuario@debian:~$ sudo nano /etc/ssh/sshd_config
```

Cambia:

```
PermitRootLogin no
```

Por:

```
PermitRootLogin yes
```

Guarda y reinicia SSH:

```bash
usuario@debian:~$ sudo systemctl restart ssh
```

### Vulnerabilidad 3 — Desactivar actualizaciones automáticas de seguridad

```bash
usuario@debian:~$ sudo apt remove unattended-upgrades -y
```

### Vulnerabilidad 4 — Instalar un servicio sin cifrar (Telnet)

```bash
usuario@debian:~$ sudo apt install telnetd -y
usuario@debian:~$ sudo systemctl start inetd
```

> ⚠️ Telnet transmite datos en texto plano. Nessus lo detectará como vulnerabilidad **High** o **Critical**.

### Vulnerabilidad 5 — Exponer servicios en puertos abiertos innecesarios

```bash
usuario@debian:~$ sudo apt install vsftpd -y
usuario@debian:~$ sudo systemctl start vsftpd
```

> ℹ️ FTP anónimo habilitado genera vulnerabilidades detectables por Nessus.

✅ **Vulnerabilidades creadas en la VM Debian.**

---

## 🔎 Punto C — Configurar y ejecutar el escaneo (Debian)

> **Objetivo:** Crear un escaneo en Nessus apuntando a la VM Debian.

### Paso 1 — Obtener la IP de la VM objetivo

En la terminal de la VM Debian:

```bash
usuario@debian:~$ ip a
```

Busca la línea `inet` bajo la interfaz de red (ej. `eth0` o `ens33`) para obtener la IP (ej. `192.168.0.15`).

### Paso 2 — Crear el escaneo en Nessus

Accede a Nessus (`https://localhost:8834`) y haz clic en **"New Scan"** → **"Basic Network Scan"**.

### Paso 3 — Configurar los parámetros

| Campo | Valor |
|---|---|
| **Name** | Escaneo Debian |
| **Description** | Escaneo de vulnerabilidades Linux |
| **Folder** | My Scans |
| **Targets** | 192.168.0.15 (IP de tu VM) |

### Paso 4 — Configurar credenciales SSH (opcional pero recomendado)

Ve a **"Credentials"** → **"SSH"** y completa:

| Campo | Valor |
|---|---|
| Authentication method | Password |
| Username | root (o usuario con sudo) |
| Password | (contraseña del sistema) |

> ℹ️ Proporcionar credenciales SSH permite a Nessus hacer un escaneo autenticado más profundo, revelando más vulnerabilidades internas del sistema.

### Paso 5 — Guardar y lanzar el escaneo

Haz clic en **"Save"** y luego en **"Launch"** (▶).

✅ **Escaneo en ejecución contra la VM Debian.**

---

## 📊 Punto D y E — Ver resultados y aplicar soluciones CVE (Debian)

> **Objetivo:** Analizar las vulnerabilidades encontradas y corregirlas.

### Paso 1 — Revisar el resumen de vulnerabilidades

Al completarse el escaneo, abre el resultado y revisa las vulnerabilidades. Un ejemplo de lo que podrías ver:

| Severidad | Vulnerabilidad típica en Debian |
|---|---|
| 🔴 Critical | Telnet habilitado (transmisión en texto plano) |
| 🟠 High | SSH permite login como root |
| 🟠 High | Firewall deshabilitado (ufw off) |
| 🟡 Medium | FTP anónimo habilitado (vsftpd) |
| 🟡 Medium | Paquetes del sistema desactualizados |

### Paso 2 — Aplicar soluciones

#### Solución para Telnet (Critical)

```bash
usuario@debian:~$ sudo apt remove telnetd -y
usuario@debian:~$ sudo systemctl stop inetd
```

#### Solución para SSH root login (High)

```bash
usuario@debian:~$ sudo nano /etc/ssh/sshd_config
```

Cambia `PermitRootLogin yes` por `PermitRootLogin no` y reinicia:

```bash
usuario@debian:~$ sudo systemctl restart ssh
```

#### Solución para el Firewall (High)

```bash
usuario@debian:~$ sudo ufw enable
usuario@debian:~$ sudo ufw default deny incoming
usuario@debian:~$ sudo ufw allow ssh
```

#### Solución para FTP anónimo (Medium)

```bash
usuario@debian:~$ sudo nano /etc/vsftpd.conf
```

Cambia:

```
anonymous_enable=YES
```

Por:

```
anonymous_enable=NO
```

Reinicia el servicio:

```bash
usuario@debian:~$ sudo systemctl restart vsftpd
```

#### Solución para paquetes desactualizados (Medium)

```bash
usuario@debian:~$ sudo apt update && sudo apt upgrade -y
```

✅ **Vulnerabilidades corregidas en la VM Debian.**

---

## 📋 Punto F — Nuevo reporte y conclusión (Debian)

> **Objetivo:** Ejecutar un segundo escaneo para verificar la reducción de vulnerabilidades.

### Paso 1 — Crear un segundo escaneo

Repite el Punto C con el mismo target pero nómbralo **"Segundo escaneo Debian"** y ejecútalo.

### Paso 2 — Comparar los resultados

| Severidad | Escaneo 1 (antes) | Escaneo 2 (después) | Reducción |
|---|---|---|---|
| 🔴 Critical | presente | 0 | ✅ eliminado |
| 🟠 High | presente | reducido | ✅ reducido |
| 🟡 Medium | presente | reducido | ✅ reducido |

### Conclusión

En entornos Linux como Debian, Nessus es igualmente efectivo para detectar configuraciones inseguras, servicios vulnerables y paquetes desactualizados. Las soluciones aplicadas (deshabilitar Telnet, bloquear acceso root SSH, activar el firewall y actualizar paquetes) demuestran que muchas vulnerabilidades críticas provienen de **malas configuraciones por defecto** más que de fallos del sistema operativo en sí. Nessus proporciona una guía clara y estructurada para resolverlas.

---

## 📊 Diferencias clave: Windows Server vs Debian

| Aspecto | Windows Server | Debian / Ubuntu |
|---|---|---|
| Instalador Nessus | `.msi` (instalador gráfico) | `.deb` (`dpkg -i`) |
| Servicio Nessus | Se gestiona automáticamente | `systemctl start nessusd` |
| Vulnerabilidades típicas | Parches KB faltantes, Defender desactivado | SSH mal configurado, Telnet, FTP anónimo |
| Autenticación en escaneo | Windows (usuario/contraseña/dominio) | SSH (usuario/contraseña o clave) |
| Solución principal | Windows Update | `apt upgrade` + correcciones manuales |
| Firewall desactivado | Windows Defender Firewall | `ufw disable` |
| CVEs comunes | Microsoft Bulletins | OpenSSH, OpenSSL, librerías del sistema |
| Tipo de escaneo recomendado | Basic Network Scan con credenciales Windows | Basic Network Scan con credenciales SSH |

---

## 📝 Resumen de Comandos (Debian)

| Comando | Descripción |
|---|---|
| `dpkg -i Nessus.deb` | Instalar Nessus en Debian |
| `systemctl start nessusd` | Iniciar el servicio de Nessus |
| `systemctl enable nessusd` | Habilitar Nessus al inicio del sistema |
| `ip a` | Ver la dirección IP del sistema |
| `ufw disable` / `ufw enable` | Desactivar / activar el firewall |
| `apt remove telnetd` | Eliminar Telnet del sistema |
| `apt update && apt upgrade -y` | Actualizar todos los paquetes del sistema |
| `nano /etc/ssh/sshd_config` | Editar configuración SSH |
| `systemctl restart ssh` | Reiniciar el servicio SSH |
| `nano /etc/vsftpd.conf` | Editar configuración FTP |

---

*Documentación realizada usando Nessus Essentials sobre Windows Server 2022 y Debian/Ubuntu*
