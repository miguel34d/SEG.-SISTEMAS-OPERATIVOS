# 🔐 Práctica 7 — Configuración de Seguridad SSH, Contraseñas y Firewall en Linux (RHEL)

> **Materia:** Seguridad de Sistemas Operativos  
> **Instructor:** Juan Alexander Ramírez  
> **Autores:** Neury Montero Mejia & Miguel Ramirez Meli
> **Fecha:** 15-3-2026

---

## 📋 Tabla de Contenidos

- [Punto A — Deshabilitar el acceso de root por SSH](#-punto-a--deshabilitar-el-acceso-de-root-por-ssh)
- [Punto B — Configurar timeout SSH en 5 minutos](#-punto-b--configurar-timeout-ssh-en-5-minutos)
- [Punto C — Política de contraseñas seguras](#-punto-c--política-de-contraseñas-seguras)
- [Punto D — Expiración de contraseñas cada 90 días](#-punto-d--expiración-de-contraseñas-cada-90-días)
- [Punto E — Configurar el Firewall (solo SSH e ICMP)](#-punto-e--configurar-el-firewall-solo-ssh-e-icmp)
- [Resumen de Comandos](#-resumen-de-comandos)

---

## 🚫 Punto A — Deshabilitar el acceso de root por SSH

> **Objetivo:** Impedir que el usuario `root` pueda conectarse remotamente vía SSH.

### Paso 1 — Abrir el Símbolo del sistema en Windows (equipo host)

En el equipo anfitrión (Windows), abre el **Símbolo del sistema** (`cmd`).

### Paso 2 — Entrar en modo root en la VM (RHEL)

En la terminal de RHEL, ejecuta `su` para obtener privilegios de superusuario:

```bash
[Neury0829@localhost ~]$ su
Password:
[root@localhost Neury0829]#
```

### Paso 3 — Obtener la dirección IP de la máquina virtual

```bash
[root@localhost Neury0829]# ifconfig
```

La IP de la máquina virtual en este caso es `192.168.0.13`. La usaremos para conectarnos vía SSH.

### Paso 4 — Conectarse vía SSH desde el equipo host (verificación previa)

Desde el **Símbolo del sistema de Windows**, ejecuta:

```
C:\Users\USER> ssh root@192.168.0.13
```

- Cuando te pida confirmar el fingerprint, escribe `yes` y presiona Enter.
- Luego ingresa la contraseña de la VM.

Una vez conectado, verifica con `whoami`:

```bash
[root@localhost ~]# whoami
root
```

> ✅ Esto confirma que SSH como root **funciona antes** del cambio. Ahora procederemos a desactivarlo.

### Paso 5 — Editar la configuración global de SSH en la VM

De vuelta en la terminal de RHEL:

```bash
[root@localhost Neury0829]# nano /etc/ssh/sshd_config
```

### Paso 6 — Encontrar y cambiar `PermitRootLogin`

Desplázate hacia abajo hasta encontrar la línea:

```
PermitRootLogin yes
```

Cámbiala a:

```
PermitRootLogin no
```

### Paso 7 — Guardar y cerrar

- Guardar: `Ctrl + O` → Enter
- Salir: `Ctrl + X`

### Paso 8 — Reiniciar el servicio SSH

```bash
[root@localhost Neury0829]# systemctl restart sshd
```

### Paso 9 — Verificar que root ya no puede conectarse

Desde el Símbolo del sistema de Windows, intenta conectarte nuevamente:

```
C:\Users\USER> ssh root@192.168.0.13
root@192.168.0.13's password:
Permission denied, please try again.
```

✅ **Aunque la contraseña sea correcta, el acceso root por SSH queda bloqueado.**

---

## ⏱️ Punto B — Configurar timeout SSH en 5 minutos

> **Objetivo:** Cerrar automáticamente las sesiones SSH inactivas tras 5 minutos (300 segundos).

### Paso 1 — Abrir nuevamente la configuración de SSH

```bash
[root@localhost Neury0829]# nano /etc/ssh/sshd_config
```

### Paso 2 — Localizar las opciones de timeout

Desplázate hacia abajo hasta encontrar estas líneas (aparecen comentadas con `#`):

```
#ClientAliveInterval 0
#ClientAliveCountMax 3
```

### Paso 3 — Descomentar y configurar los valores

Elimina el `#` de ambas líneas y ajusta los valores así:

```
ClientAliveInterval 300
ClientAliveCountMax 0
```

| Parámetro | Valor | Descripción |
|---|---|---|
| `ClientAliveInterval` | `300` | Envía una señal cada 300 segundos (5 minutos) de inactividad |
| `ClientAliveCountMax` | `0` | Si no hay respuesta, cierra la sesión inmediatamente |

### Paso 4 — Guardar, cerrar y reiniciar el servicio

```
Ctrl + O → Enter → Ctrl + X
```

```bash
[root@localhost Neury0829]# systemctl restart sshd
```

✅ **Las sesiones SSH inactivas por más de 5 minutos se cerrarán automáticamente.**

---

## 🔑 Punto C — Política de contraseñas seguras

> **Objetivo:** Configurar que toda contraseña requiera: mínimo 8 caracteres, una mayúscula, una minúscula, un símbolo y un número.

### Paso 1 — Abrir el archivo de políticas de contraseñas

```bash
[root@localhost Neury0829]# nano /etc/security/pwquality.conf
```

### Paso 2 — Configurar longitud mínima

Busca la línea `# minlen = 8`, descoméntala:

```
minlen = 8
```

### Paso 3 — Requerir al menos una mayúscula

Busca `# ucredit`, descoméntala y asígnale el valor `1`:

```
ucredit = 1
```

### Paso 4 — Requerir al menos una minúscula

Busca `# lcredit`, descoméntala y asígnale el valor `1`:

```
lcredit = 1
```

### Paso 5 — Requerir al menos un símbolo

Busca `# ocredit`, descoméntala y asígnale el valor `1`:

```
ocredit = 1
```

### Paso 6 — Requerir al menos una clase de carácter obligatoria

Busca `# minclass`, descoméntala y asígnale el valor `1`:

```
minclass = 1
```

> ℹ️ `minclass` obliga a que la contraseña tenga al menos un carácter de cada clase (dígito, mayúscula, minúscula, símbolo).

### Paso 7 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

✅ **Las nuevas contraseñas deberán cumplir todos los requisitos configurados.**

---

## 📅 Punto D — Expiración de contraseñas cada 90 días

> **Objetivo:** Forzar que las contraseñas del sistema expiren automáticamente cada 90 días.

### Método 1 — Configuración global (para todos los usuarios nuevos)

#### Paso 1 — Abrir el archivo de configuración de login

```bash
[root@localhost Neury0829]# nano /etc/login.defs
```

#### Paso 2 — Encontrar y modificar `PASS_MAX_DAYS`

Desplázate hasta encontrar:

```
PASS_MAX_DAYS   99999
```

Cámbialo a:

```
PASS_MAX_DAYS   90
```

| Parámetro | Valor anterior | Valor nuevo |
|---|---|---|
| `PASS_MAX_DAYS` | `99999` (sin expiración) | `90` (90 días) |
| `PASS_MIN_DAYS` | `0` | `0` (sin cambio) |
| `PASS_MIN_LEN` | `5` | `5` (sin cambio) |
| `PASS_WARN_AGE` | `7` | `7` (aviso 7 días antes) |

#### Paso 3 — Guardar y cerrar

```
Ctrl + O → Enter → Ctrl + X
```

---

### Método 2 — Configuración por usuario específico

Para aplicar la expiración a un usuario ya existente (ej. `nmontero`):

```bash
[root@localhost Neury0829]# chage -M 90 nmontero
```

Para verificar la configuración del usuario:

```bash
[root@localhost Neury0829]# chage -l nmontero
```

Resultado esperado:

```
Last password change          : Mar 04, 2026
Password expires              : Jun 02, 2026
Password inactive             : never
Account expires               : never
Minimum number of days between password change   : 0
Maximum number of days between password change   : 90
Number of days of warning before password expires: 7
```

✅ **Las contraseñas expirarán a los 90 días de su último cambio.**

---

## 🛡️ Punto E — Configurar el Firewall (solo SSH e ICMP)

> **Objetivo:** Configurar `firewalld` para permitir únicamente tráfico SSH y paquetes ICMP; bloquear todo lo demás.

### Paso 1 — Listar los servicios actualmente habilitados

```bash
[root@localhost Neury0829]# firewall-cmd --list-service
cockpit dhcpv6-client ssh
```

### Paso 2 — Agregar SSH de forma permanente

```bash
[root@localhost Neury0829]# firewall-cmd --add-service=ssh --permanent
Warning: ALREADY_ENABLED: ssh
success
```

> ℹ️ La advertencia `ALREADY_ENABLED` indica que SSH ya estaba habilitado. Esto es normal.

### Paso 3 — Habilitar los paquetes ICMP (ping)

```bash
[root@localhost Neury0829]# firewall-cmd --add-icmp-block-inversion --permanent
success
```

> ℹ️ `--add-icmp-block-inversion` invierte el comportamiento por defecto: en lugar de bloquear tipos ICMP específicos, **permite** los que no estén bloqueados (permitiendo el ping).

### Paso 4 — Recargar el firewall para aplicar los cambios

```bash
[root@localhost Neury0829]# firewall-cmd --reload
success
```

✅ **El firewall ahora permite únicamente SSH e ICMP. El resto del tráfico está bloqueado.**

---

## 📝 Resumen de Comandos

| Comando | Descripción |
|---|---|
| `nano /etc/ssh/sshd_config` | Editar configuración del servidor SSH |
| `systemctl restart sshd` | Reiniciar el servicio SSH |
| `nano /etc/security/pwquality.conf` | Editar política de calidad de contraseñas |
| `nano /etc/login.defs` | Editar configuración de expiración de contraseñas |
| `chage -M 90 <usuario>` | Establecer expiración de 90 días para un usuario |
| `chage -l <usuario>` | Ver configuración de contraseña de un usuario |
| `firewall-cmd --list-service` | Listar servicios habilitados en el firewall |
| `firewall-cmd --add-service=ssh --permanent` | Permitir SSH de forma permanente |
| `firewall-cmd --add-icmp-block-inversion --permanent` | Permitir tráfico ICMP |
| `firewall-cmd --reload` | Aplicar cambios en el firewall |
| `ifconfig` | Ver la dirección IP de la máquina |
| `ssh <usuario>@<ip>` | Conectarse vía SSH a un equipo remoto |
| `whoami` | Verificar el usuario activo actual |

---

### 🔐 Parámetros clave en `/etc/ssh/sshd_config`

| Parámetro | Valor | Efecto |
|---|---|---|
| `PermitRootLogin` | `no` | Bloquea el acceso SSH como root |
| `ClientAliveInterval` | `300` | Envía señal cada 5 min de inactividad |
| `ClientAliveCountMax` | `0` | Cierra la sesión si no hay respuesta |

### 🔒 Parámetros clave en `/etc/security/pwquality.conf`

| Parámetro | Valor | Efecto |
|---|---|---|
| `minlen` | `8` | Mínimo 8 caracteres |
| `ucredit` | `1` | Al menos 1 letra mayúscula |
| `lcredit` | `1` | Al menos 1 letra minúscula |
| `ocredit` | `1` | Al menos 1 símbolo especial |
| `minclass` | `1` | Al menos 1 clase de carácter obligatoria |

---

*Documentación realizada en RHEL (Red Hat Enterprise Linux)*

---

# 🐧 VERSIÓN DEBIAN

> Los mismos objetivos de la práctica, adaptados para distribuciones basadas en **Debian/Ubuntu**.  
> Los archivos de configuración y algunos comandos difieren de RHEL.

---

## 🚫 Punto A — Deshabilitar el acceso de root por SSH (Debian)

### Paso 1 — Entrar en modo root

```bash
usuario@debian:~$ sudo su
root@debian:/home/usuario#
```

### Paso 2 — Obtener la IP de la VM

```bash
root@debian:~# ip a
```

Busca la línea `inet` bajo la interfaz de red (ej. `eth0` o `ens33`) para obtener la IP.

### Paso 3 — Editar la configuración SSH

```bash
root@debian:~# nano /etc/ssh/sshd_config
```

Busca y cambia:

```
PermitRootLogin yes
```

Por:

```
PermitRootLogin no
```

### Paso 4 — Reiniciar el servicio SSH

```bash
root@debian:~# systemctl restart ssh
```

> ⚠️ En Debian/Ubuntu el servicio se llama `ssh`, no `sshd`.

✅ **Root ya no podrá conectarse vía SSH.**

---

## ⏱️ Punto B — Configurar timeout SSH en 5 minutos (Debian)

### Paso 1 — Editar la configuración SSH

```bash
root@debian:~# nano /etc/ssh/sshd_config
```

### Paso 2 — Configurar los parámetros de timeout

Busca y edita (o agrega si no existen):

```
ClientAliveInterval 300
ClientAliveCountMax 0
```

### Paso 3 — Reiniciar el servicio

```bash
root@debian:~# systemctl restart ssh
```

✅ **Las sesiones inactivas se cerrarán tras 5 minutos.**

---

## 🔑 Punto C — Política de contraseñas seguras (Debian)

En Debian/Ubuntu se usa el paquete `libpam-pwquality`. Instálalo si no está presente:

```bash
root@debian:~# apt install libpam-pwquality -y
```

### Paso 1 — Editar el archivo de políticas

```bash
root@debian:~# nano /etc/security/pwquality.conf
```

### Paso 2 — Configurar los parámetros

Descomenta y ajusta los siguientes valores:

```
minlen = 8
ucredit = 1
lcredit = 1
ocredit = 1
minclass = 1
```

> ℹ️ El archivo y los parámetros son **idénticos** a los de RHEL. Solo cambia el método de instalación del paquete.

✅ **Las contraseñas deberán cumplir los requisitos configurados.**

---

## 📅 Punto D — Expiración de contraseñas cada 90 días (Debian)

### Método 1 — Configuración global

```bash
root@debian:~# nano /etc/login.defs
```

Busca y cambia:

```
PASS_MAX_DAYS   99999
```

Por:

```
PASS_MAX_DAYS   90
```

> ℹ️ El archivo `/etc/login.defs` existe en Debian también y funciona de la misma manera.

### Método 2 — Por usuario específico

```bash
root@debian:~# chage -M 90 nombre_usuario
root@debian:~# chage -l nombre_usuario
```

✅ **Las contraseñas expirarán a los 90 días.**

---

## 🛡️ Punto E — Configurar el Firewall (solo SSH e ICMP) en Debian

En Debian/Ubuntu se usa **`ufw`** (Uncomplicated Firewall) en lugar de `firewalld`.

### Paso 1 — Instalar ufw (si no está instalado)

```bash
root@debian:~# apt install ufw -y
```

### Paso 2 — Restablecer las reglas a estado limpio

```bash
root@debian:~# ufw reset
```

### Paso 3 — Establecer política por defecto: bloquear todo

```bash
root@debian:~# ufw default deny incoming
root@debian:~# ufw default allow outgoing
```

### Paso 4 — Permitir SSH

```bash
root@debian:~# ufw allow ssh
```

### Paso 5 — Permitir paquetes ICMP (ping)

Edita el archivo de reglas antes de activar `ufw`:

```bash
root@debian:~# nano /etc/ufw/before.rules
```

Busca el bloque que empieza con `# ok icmp codes for INPUT` y verifica que estas líneas **no** estén comentadas:

```
-A ufw-before-input -p icmp --icmp-type echo-request -j ACCEPT
```

### Paso 6 — Activar el firewall

```bash
root@debian:~# ufw enable
```

### Paso 7 — Verificar las reglas activas

```bash
root@debian:~# ufw status verbose
```

Resultado esperado:

```
Status: active
To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere   (SSH)
```

✅ **Solo SSH e ICMP están permitidos. Todo lo demás queda bloqueado.**

---

## 📝 Diferencias clave: RHEL vs Debian

| Aspecto | RHEL / CentOS | Debian / Ubuntu |
|---|---|---|
| Servicio SSH | `sshd` | `ssh` |
| Reiniciar SSH | `systemctl restart sshd` | `systemctl restart ssh` |
| Firewall | `firewalld` / `firewall-cmd` | `ufw` |
| Instalar paquetes | `dnf install` | `apt install` |
| Política de contraseñas | `pwquality` (incluido) | `libpam-pwquality` (instalar) |
| Archivo `login.defs` | `/etc/login.defs` | `/etc/login.defs` (igual) |
| Archivo SSH config | `/etc/ssh/sshd_config` | `/etc/ssh/sshd_config` (igual) |
| Archivo pwquality | `/etc/security/pwquality.conf` | `/etc/security/pwquality.conf` (igual) |

---

*Documentación realizada en RHEL (Red Hat Enterprise Linux) y Debian/Ubuntu*
