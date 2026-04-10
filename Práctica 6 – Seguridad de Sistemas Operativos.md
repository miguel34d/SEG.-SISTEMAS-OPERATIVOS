[Práctica_6_RHEL_y_Debian.md](https://github.com/user-attachments/files/26629251/Practica_6_RHEL_y_Debian.md)
# 🔐 Práctica 6 — Gestión de Usuarios y Grupos en Linux

> **Materia:** Seguridad de Sistemas Operativos  
> **Instructor:** Juan Alexander Ramírez  
> **Autores:** Neury Montero Mejia & Miguel Ramirez Meli 
> **Institución:** Instituto Tecnológico de Las Américas (ITLA)  
> **Fecha:** 8-3-2026

---

## 📋 Tabla de Contenidos

### 🔴 Versión RHEL
- [Punto A — Crear 6 usuarios (RHEL)](#-punto-a--crear-6-usuarios)
- [Punto B — Crear grupos y añadir usuarios (RHEL)](#-punto-b--crear-grupos-y-añadir-usuarios)
- [Punto C — Crear usuario "profesor" y grupo "docentes" (RHEL)](#-punto-c--crear-usuario-profesor-y-grupo-docentes)
- [Punto D — Habilitar privilegios sudo (RHEL)](#-punto-d--habilitar-privilegios-sudo)
- [Punto E — Retirar usuarios del grupo prueba y eliminar el grupo (RHEL)](#-punto-e--retirar-usuarios-del-grupo-prueba-y-eliminar-el-grupo)
- [Punto F — Eliminar un usuario (RHEL)](#-punto-f--eliminar-un-usuario)
- [Punto G — Añadir usuario al grupo docentes y verificar privilegios sudo (RHEL)](#-punto-g--añadir-usuario-al-grupo-docentes-y-verificar-privilegios-sudo)
- [Punto I — Cambiar contraseñas (RHEL)](#-punto-i--cambiar-contraseñas)

### 🐧 Versión Debian
- [Punto A — Crear 6 usuarios (Debian)](#-punto-a--crear-6-usuarios-debian)
- [Punto B — Crear grupos y añadir usuarios (Debian)](#-punto-b--crear-grupos-y-añadir-usuarios-debian)
- [Punto C — Crear usuario "profesor" y grupo "docentes" (Debian)](#-punto-c--crear-usuario-profesor-y-grupo-docentes-debian)
- [Punto D — Habilitar privilegios sudo (Debian)](#-punto-d--habilitar-privilegios-sudo-debian)
- [Punto E — Retirar usuarios del grupo prueba y eliminar el grupo (Debian)](#-punto-e--retirar-usuarios-del-grupo-prueba-y-eliminar-el-grupo-debian)
- [Punto F — Eliminar un usuario (Debian)](#-punto-f--eliminar-un-usuario-debian)
- [Punto G — Añadir usuario al grupo docentes y verificar privilegios sudo (Debian)](#-punto-g--añadir-usuario-al-grupo-docentes-y-verificar-privilegios-sudo-debian)
- [Punto I — Cambiar contraseñas (Debian)](#-punto-i--cambiar-contraseñas-debian)

### 📊 [Diferencias clave: RHEL vs Debian](#-diferencias-clave-rhel-vs-debian)

- [Punto G — Añadir usuario al grupo docentes y verificar privilegios sudo](#-punto-g--añadir-usuario-al-grupo-docentes-y-verificar-privilegios-sudo)
- [Punto I — Cambiar contraseñas](#-punto-i--cambiar-contraseñas)

---

## 👥 Punto A — Crear 6 usuarios

> **Objetivo:** Crear 6 usuarios siguiendo la nomenclatura: primera letra del nombre + apellido.

### Paso 1 — Entrar en modo root

Abre la terminal y ejecuta el comando `su`. Ingresa la contraseña cuando se solicite:

```bash
[Neury0829@localhost ~]$ su
Password:
[root@localhost Neury0829]#
```

### Paso 2 — Crear el primer usuario

Usa el comando `useradd` con el flag `-m` para crear automáticamente un directorio home:

```bash
[root@localhost Neury0829]# useradd -m nmontero
```

| Parte del comando | Descripción |
|---|---|
| `useradd` | Comando para crear usuarios |
| `-m` | Crea un directorio `/home` para el usuario |
| `nmontero` | Nombre del usuario a crear |

### Paso 3 — Repetir para los 6 usuarios

Usa la flecha ↑ del teclado para reutilizar el comando y cambia el nombre del usuario:

```bash
[root@localhost Neury0829]# useradd -m nmontero
[root@localhost Neury0829]# useradd -m amejia
[root@localhost Neury0829]# useradd -m jmontero
[root@localhost Neury0829]# useradd -m jmarte
[root@localhost Neury0829]# useradd -m jirrizarri
[root@localhost Neury0829]# useradd -m smontero
```

### Paso 4 — Verificar la creación de usuarios

Confirma que los usuarios se crearon correctamente:

```bash
[root@localhost Neury0829]# cat /etc/passwd
```

Los nuevos usuarios aparecerán al final de la lista:

```
nmontero:x:1001:1001::/home/nmontero:/bin/bash
amejia:x:1002:1002::/home/amejia:/bin/bash
jmontero:x:1003:1003::/home/jmontero:/bin/bash
jmarte:x:1004:1004::/home/jmarte:/bin/bash
jirrizarri:x:1005:1005::/home/jirrizarri:/bin/bash
smontero:x:1006:1006::/home/smontero:/bin/bash
```

✅ **Los usuarios se han creado satisfactoriamente.**

---

## 🗂️ Punto B — Crear grupos y añadir usuarios

> **Objetivo:** Crear los grupos `estudiantes` y `prueba`, luego añadir todos los usuarios a ambos grupos.

### Paso 1 — Crear los grupos

```bash
[root@localhost Neury0829]# groupadd estudiantes
[root@localhost Neury0829]# groupadd pruebas
```

### Paso 2 — Verificar que los grupos se crearon

```bash
[root@localhost Neury0829]# cat /etc/group
```

Al final del listado deberás ver:

```
estudiantes:x:1007:
pruebas:x:1008:
```

✅ **Grupos creados exitosamente.**

### Paso 3 — Añadir usuarios al grupo `estudiantes`

Usa `usermod -aG` para agregar cada usuario como miembro **secundario** del grupo (el flag `-aG` evita conflictos con el grupo primario):

```bash
[root@localhost Neury0829]# usermod -aG estudiantes nmontero
[root@localhost Neury0829]# usermod -aG estudiantes amejia
[root@localhost Neury0829]# usermod -aG estudiantes jmontero
[root@localhost Neury0829]# usermod -aG estudiantes jmarte
[root@localhost Neury0829]# usermod -aG estudiantes jirrizarri
[root@localhost Neury0829]# usermod -aG estudiantes smontero
```

### Paso 4 — Añadir usuarios al grupo `pruebas`

Repite el proceso cambiando el nombre del grupo:

```bash
[root@localhost Neury0829]# usermod -aG pruebas smontero
[root@localhost Neury0829]# usermod -aG pruebas nmontero
[root@localhost Neury0829]# usermod -aG pruebas amejia
[root@localhost Neury0829]# usermod -aG pruebas jmontero
[root@localhost Neury0829]# usermod -aG pruebas jmarte
[root@localhost Neury0829]# usermod -aG pruebas jirrizarri
```

### Paso 5 — Verificar la incorporación de usuarios

```bash
[root@localhost Neury0829]# cat /etc/group
```

Resultado esperado:

```
estudiantes:x:1007:nmontero,amejia,jmontero,jmarte,jirrizarri,smontero
pruebas:x:1008:smontero,nmontero,amejia,jmontero,jmarte,jirrizarri
```

✅ **Todos los usuarios se han agregado con éxito a ambos grupos.**

---

## 🧑‍🏫 Punto C — Crear usuario "profesor" y grupo "docentes"

> **Objetivo:** Crear un nuevo usuario llamado `profesor` y un grupo llamado `docentes`.

### Paso 1 — Crear el usuario y el grupo

```bash
[root@localhost Neury0829]# useradd -m profesor
[root@localhost Neury0829]# groupadd docentes
```

### Paso 2 — Verificar el grupo

```bash
[root@localhost Neury0829]# cat /etc/group
```

Deberás ver al final:

```
profesor:x:1009:
docentes:x:1010:
```

### Paso 3 — Verificar el usuario

```bash
[root@localhost Neury0829]# cat /etc/passwd
```

Deberás ver al final:

```
profesor:x:1007:1009::/home/profesor:/bin/bash
```

✅ **Usuario y grupo creados satisfactoriamente.**

---

## 🔑 Punto D — Habilitar privilegios sudo

> **Objetivo:** Editar `/etc/sudoers` para dar privilegios `sudo` al usuario `profesor` y al grupo `docentes`.

### Paso 1 — Abrir el archivo sudoers con nano

```bash
[root@localhost Neury0829]# nano /etc/sudoers
```

Esto abrirá el editor de texto `nano` con la configuración de sudoers.

### Paso 2 — Agregar las entradas al final del archivo

Navega hasta el final del archivo y añade las siguientes dos líneas:

```
profesor    ALL=(ALL)    ALL
%docentes   ALL=(ALL)    ALL
```

> ⚠️ **Importante:** El símbolo `%` antes del nombre indica que es un **grupo**, no un usuario. `ALL=(ALL) ALL` otorga permisos de sudo completos.

### Paso 3 — Guardar y cerrar

- Guardar: `Ctrl + O` → Enter
- Salir: `Ctrl + X`

### Paso 4 — Verificar la configuración guardada

```bash
[root@localhost Neury0829]# cat /etc/sudoers
```

Al final del archivo deberás ver:

```
profesor ALL=(ALL)    ALL
%docentes ALL=(ALL)   ALL
```

✅ **Configuración guardada correctamente.**

---

## 🗑️ Punto E — Retirar usuarios del grupo prueba y eliminar el grupo

> **Objetivo:** Remover a todos los usuarios del grupo `pruebas` y luego eliminar el grupo.

### Paso 1 — Retirar usuarios del grupo `pruebas`

Usa el comando `gpasswd -d` (donde `-d` significa *delete*):

```bash
[root@localhost Neury0829]# gpasswd -d nmontero pruebas
Removing user nmontero from group pruebas

[root@localhost Neury0829]# gpasswd -d amejia pruebas
Removing user amejia from group pruebas

[root@localhost Neury0829]# gpasswd -d jmarte pruebas
Removing user jmarte from group pruebas

[root@localhost Neury0829]# gpasswd -d smontero pruebas
Removing user smontero from group pruebas

[root@localhost Neury0829]# gpasswd -d jmontero pruebas
Removing user jmontero from group pruebas

[root@localhost Neury0829]# gpasswd -d jirrizarri pruebas
Removing user jirrizarri from group pruebas
```

### Paso 2 — Eliminar el grupo `pruebas`

```bash
[root@localhost Neury0829]# groupdel pruebas
```

### Paso 3 — Verificar la eliminación

```bash
[root@localhost Neury0829]# cat /etc/group
```

El grupo `pruebas` ya no debe aparecer en la lista.

✅ **Grupo eliminado correctamente.**

---

## ❌ Punto F — Eliminar un usuario

> **Objetivo:** Eliminar uno de los 6 usuarios creados (en este caso, `jirrizarri`).

### Paso 1 — Eliminar el usuario con `userdel`

```bash
[root@localhost Neury0829]# userdel jirrizarri
```

### Paso 2 — Verificar la eliminación

```bash
[root@localhost Neury0829]# cat /etc/passwd
```

El usuario `jirrizarri` ya no debe aparecer en el listado.

✅ **El usuario `jirrizarri` fue eliminado exitosamente.**

---

## 🛡️ Punto G — Añadir usuario al grupo docentes y verificar privilegios sudo

> **Objetivo:** Agregar `nmontero` al grupo `docentes` y comprobar que hereda los privilegios sudo configurados previamente.

### Paso 1 — Añadir el usuario al grupo `docentes`

```bash
[root@localhost Neury0829]# usermod -aG nmontero docentes
```

### Paso 2 — Verificar la incorporación

```bash
[root@localhost Neury0829]# cat /etc/group
```

Resultado esperado:

```
docentes:x:1010:nmontero
```

✅ **Usuario incorporado al grupo.**

### Paso 3 — Asignar contraseña al usuario `nmontero`

```bash
[root@localhost Neury0829]# passwd nmontero
Changing password for user nmontero.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

### Paso 4 — Acceder como `nmontero`

```bash
[root@localhost Neury0829]# su - nmontero
[nmontero@localhost ~]$
```

✅ **Acceso exitoso al usuario.**

### Paso 5 — Probar los privilegios sudo

```bash
[nmontero@localhost ~]$ sudo su
[sudo] password for nmontero:
[root@localhost nmontero]#
```

### Paso 6 — Confirmar con `whoami`

```bash
[root@localhost nmontero]# whoami
root
```

✅ **El usuario `nmontero` tiene privilegios sudo gracias a pertenecer al grupo `docentes`.**

---

## 🔒 Punto I — Cambiar contraseñas de los usuarios

> **Objetivo:** Cambiar la contraseña de todos los usuarios creados anteriormente.

### Paso 1 — Usar `passwd` para cada usuario

```bash
[root@localhost nmontero]# passwd amejia
Changing password for user amejia.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

[root@localhost nmontero]# passwd jmarte
Changing password for user jmarte.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

[root@localhost nmontero]# passwd jmontero
Changing password for user jmontero.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.

[root@localhost nmontero]# passwd smontero
Changing password for user smontero.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

✅ **Contraseñas actualizadas satisfactoriamente para todos los usuarios.**

---

## 📝 Resumen de Comandos Utilizados

| Comando | Descripción |
|---|---|
| `su` | Cambiar a usuario root |
| `useradd -m <usuario>` | Crear un usuario con directorio home |
| `userdel <usuario>` | Eliminar un usuario |
| `groupadd <grupo>` | Crear un grupo |
| `groupdel <grupo>` | Eliminar un grupo |
| `usermod -aG <grupo> <usuario>` | Añadir usuario a un grupo secundario |
| `gpasswd -d <usuario> <grupo>` | Retirar un usuario de un grupo |
| `passwd <usuario>` | Cambiar la contraseña de un usuario |
| `cat /etc/passwd` | Listar todos los usuarios del sistema |
| `cat /etc/group` | Listar todos los grupos del sistema |
| `cat /etc/sudoers` | Ver configuración de privilegios sudo |
| `nano /etc/sudoers` | Editar la configuración de sudoers |
| `sudo su` | Escalar privilegios a root |
| `whoami` | Ver el usuario actual activo |

---

*Documentación realizada en RHEL (Red Hat Enterprise Linux)*

---
---

# 🐧 VERSIÓN DEBIAN

> Los mismos objetivos de la práctica, adaptados para **Debian/Ubuntu**.  
> Los comandos son idénticos a RHEL en su mayoría. Las diferencias están señaladas.

---

## 👥 Punto A — Crear 6 usuarios (Debian)

> **Objetivo:** Crear 6 usuarios siguiendo la nomenclatura: primera letra del nombre + apellido.

### Paso 1 — Entrar en modo root

```bash
usuario@debian:~$ sudo su
root@debian:/home/usuario#
```

> ℹ️ En Debian se usa `sudo su` en lugar de `su` directamente, ya que la cuenta root suele estar bloqueada por defecto.

### Paso 2 — Crear los 6 usuarios

```bash
root@debian:~# useradd -m nmontero
root@debian:~# useradd -m amejia
root@debian:~# useradd -m jmontero
root@debian:~# useradd -m jmarte
root@debian:~# useradd -m jirrizarri
root@debian:~# useradd -m smontero
```

### Paso 3 — Verificar la creación de usuarios

```bash
root@debian:~# cat /etc/passwd
```

Los nuevos usuarios aparecerán al final:

```
nmontero:x:1001:1001::/home/nmontero:/bin/sh
amejia:x:1002:1002::/home/amejia:/bin/sh
jmontero:x:1003:1003::/home/jmontero:/bin/sh
jmarte:x:1004:1004::/home/jmarte:/bin/sh
jirrizarri:x:1005:1005::/home/jirrizarri:/bin/sh
smontero:x:1006:1006::/home/smontero:/bin/sh
```

> ℹ️ En Debian el shell por defecto es `/bin/sh` en lugar de `/bin/bash`. Para usar bash puedes agregar `-s /bin/bash` al comando: `useradd -m -s /bin/bash nmontero`

✅ **Usuarios creados satisfactoriamente.**

---

## 🗂️ Punto B — Crear grupos y añadir usuarios (Debian)

> **Objetivo:** Crear los grupos `estudiantes` y `pruebas`, luego añadir todos los usuarios a ambos grupos.

### Paso 1 — Crear los grupos

```bash
root@debian:~# groupadd estudiantes
root@debian:~# groupadd pruebas
```

### Paso 2 — Verificar que los grupos se crearon

```bash
root@debian:~# cat /etc/group
```

### Paso 3 — Añadir usuarios al grupo `estudiantes`

```bash
root@debian:~# usermod -aG estudiantes nmontero
root@debian:~# usermod -aG estudiantes amejia
root@debian:~# usermod -aG estudiantes jmontero
root@debian:~# usermod -aG estudiantes jmarte
root@debian:~# usermod -aG estudiantes jirrizarri
root@debian:~# usermod -aG estudiantes smontero
```

### Paso 4 — Añadir usuarios al grupo `pruebas`

```bash
root@debian:~# usermod -aG pruebas smontero
root@debian:~# usermod -aG pruebas nmontero
root@debian:~# usermod -aG pruebas amejia
root@debian:~# usermod -aG pruebas jmontero
root@debian:~# usermod -aG pruebas jmarte
root@debian:~# usermod -aG pruebas jirrizarri
```

### Paso 5 — Verificar la incorporación

```bash
root@debian:~# cat /etc/group
```

Resultado esperado:

```
estudiantes:x:1007:nmontero,amejia,jmontero,jmarte,jirrizarri,smontero
pruebas:x:1008:smontero,nmontero,amejia,jmontero,jmarte,jirrizarri
```

✅ **Todos los usuarios añadidos con éxito a ambos grupos.**

---

## 🧑‍🏫 Punto C — Crear usuario "profesor" y grupo "docentes" (Debian)

> **Objetivo:** Crear un nuevo usuario llamado `profesor` y un grupo llamado `docentes`.

### Paso 1 — Crear el usuario y el grupo

```bash
root@debian:~# useradd -m -s /bin/bash profesor
root@debian:~# groupadd docentes
```

### Paso 2 — Verificar

```bash
root@debian:~# cat /etc/group
root@debian:~# cat /etc/passwd
```

✅ **Usuario y grupo creados satisfactoriamente.**

---

## 🔑 Punto D — Habilitar privilegios sudo (Debian)

> **Objetivo:** Dar privilegios `sudo` al usuario `profesor` y al grupo `docentes`.

### Opción 1 — Editar `/etc/sudoers` con visudo (recomendado en Debian)

En Debian se recomienda usar `visudo` en lugar de `nano` directamente, ya que valida la sintaxis antes de guardar:

```bash
root@debian:~# visudo
```

> ℹ️ `visudo` abre el archivo con el editor por defecto (generalmente `nano` en Debian).

### Paso 2 — Agregar las entradas al final del archivo

```
profesor    ALL=(ALL)    ALL
%docentes   ALL=(ALL)    ALL
```

> ⚠️ El símbolo `%` antes del nombre indica que es un **grupo**.

### Opción 2 — Agregar al grupo sudo directamente (método Debian nativo)

En Debian, también puedes simplemente agregar el usuario al grupo `sudo`, que ya tiene permisos completos:

```bash
root@debian:~# usermod -aG sudo profesor
```

Para agregar a todos los miembros del grupo `docentes` al grupo `sudo`:

```bash
root@debian:~# usermod -aG sudo nmontero
```

### Paso 3 — Verificar

```bash
root@debian:~# cat /etc/sudoers
```

✅ **Privilegios sudo configurados.**

---

## 🗑️ Punto E — Retirar usuarios del grupo prueba y eliminar el grupo (Debian)

> **Objetivo:** Remover a todos los usuarios del grupo `pruebas` y luego eliminar el grupo.

### Paso 1 — Retirar usuarios del grupo `pruebas`

```bash
root@debian:~# gpasswd -d nmontero pruebas
Removing user nmontero from group pruebas

root@debian:~# gpasswd -d amejia pruebas
Removing user amejia from group pruebas

root@debian:~# gpasswd -d jmarte pruebas
Removing user jmarte from group pruebas

root@debian:~# gpasswd -d smontero pruebas
Removing user smontero from group pruebas

root@debian:~# gpasswd -d jmontero pruebas
Removing user jmontero from group pruebas

root@debian:~# gpasswd -d jirrizarri pruebas
Removing user jirrizarri from group pruebas
```

### Paso 2 — Eliminar el grupo `pruebas`

```bash
root@debian:~# groupdel pruebas
```

### Paso 3 — Verificar la eliminación

```bash
root@debian:~# cat /etc/group
```

✅ **Grupo eliminado correctamente.**

---

## ❌ Punto F — Eliminar un usuario (Debian)

> **Objetivo:** Eliminar uno de los 6 usuarios creados.

### Paso 1 — Eliminar el usuario

```bash
root@debian:~# userdel jirrizarri
```

Para eliminar también su directorio home:

```bash
root@debian:~# userdel -r jirrizarri
```

> ℹ️ En Debian se recomienda usar `-r` para limpiar también el directorio `/home/jirrizarri` y evitar archivos huérfanos.

### Paso 2 — Verificar la eliminación

```bash
root@debian:~# cat /etc/passwd
```

✅ **El usuario `jirrizarri` fue eliminado exitosamente.**

---

## 🛡️ Punto G — Añadir usuario al grupo docentes y verificar privilegios sudo (Debian)

> **Objetivo:** Agregar `nmontero` al grupo `docentes` y comprobar que hereda los privilegios sudo.

### Paso 1 — Añadir el usuario al grupo `docentes`

```bash
root@debian:~# usermod -aG docentes nmontero
```

### Paso 2 — Verificar la incorporación

```bash
root@debian:~# cat /etc/group
```

Resultado esperado:

```
docentes:x:1010:nmontero
```

### Paso 3 — Asignar contraseña al usuario `nmontero`

```bash
root@debian:~# passwd nmontero
New password:
Retype new password:
passwd: password updated successfully.
```

### Paso 4 — Acceder como `nmontero`

```bash
root@debian:~# su - nmontero
nmontero@debian:~$
```

### Paso 5 — Probar los privilegios sudo

```bash
nmontero@debian:~$ sudo su
[sudo] password for nmontero:
root@debian:/home/nmontero#
```

### Paso 6 — Confirmar con `whoami`

```bash
root@debian:/home/nmontero# whoami
root
```

✅ **El usuario `nmontero` tiene privilegios sudo gracias a pertenecer al grupo `docentes`.**

---

## 🔒 Punto I — Cambiar contraseñas de los usuarios (Debian)

> **Objetivo:** Cambiar la contraseña de todos los usuarios creados.

### Paso 1 — Usar `passwd` para cada usuario

```bash
root@debian:~# passwd amejia
New password:
Retype new password:
passwd: password updated successfully.

root@debian:~# passwd jmarte
New password:
Retype new password:
passwd: password updated successfully.

root@debian:~# passwd jmontero
New password:
Retype new password:
passwd: password updated successfully.

root@debian:~# passwd smontero
New password:
Retype new password:
passwd: password updated successfully.
```

✅ **Contraseñas actualizadas satisfactoriamente.**

---

## 📊 Diferencias clave: RHEL vs Debian

| Aspecto | RHEL / CentOS | Debian / Ubuntu |
|---|---|---|
| Entrar como root | `su` | `sudo su` |
| Shell por defecto al crear usuario | `/bin/bash` | `/bin/sh` (usar `-s /bin/bash` para bash) |
| Eliminar usuario con home | `userdel <usuario>` | `userdel -r <usuario>` (recomendado) |
| Editar sudoers | `nano /etc/sudoers` | `visudo` (valida sintaxis) |
| Método nativo sudo | Editar `/etc/sudoers` | Agregar al grupo `sudo` con `usermod -aG sudo` |
| Mensaje al cambiar contraseña | `all authentication tokens updated successfully` | `password updated successfully` |
| Archivos de usuarios | `/etc/passwd`, `/etc/group`, `/etc/sudoers` | Idénticos |
| Comandos `useradd`, `groupadd`, `usermod`, `gpasswd`, `groupdel`, `passwd` | Igual en ambas distros | Igual en ambas distros |

---

*Documentación realizada en RHEL (Red Hat Enterprise Linux) y Debian/Ubuntu*
