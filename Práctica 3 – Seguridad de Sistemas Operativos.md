[Practica3_Seguridad_SO.md](https://github.com/user-attachments/files/26659409/Practica3_Seguridad_SO.md)
<div align="center">

# 🔐 Práctica 3 – Seguridad de Sistemas Operativos

**Instituto Tecnológico de Las Américas (ITLA)**

| Campo | Detalle |
|---|---|
| **Materia** | Seguridad de Sistemas Operativos |
| **Instructor** | Juan Alexander Ramírez |
| **Autores** | Neury Montero Mejia & Miguel Ramirez Meli |
| **Fecha** | 08-02-2026 |

</div>

---

## 📋 Tabla de Contenido

- [🛡️ Punto A – GPO Credential Guard: habilitar seguridad basada en virtualización](#punto-a--gpo-credential-guard-habilitar-seguridad-basada-en-virtualización)
- [⚙️ Punto B – Prerrequisitos de la VM cliente para Credential Guard](#punto-b--prerrequisitos-de-la-vm-cliente-para-credential-guard)
- [🚫 Punto C – GPO Block USB: bloquear dispositivos de almacenamiento USB](#punto-c--gpo-block-usb-bloquear-dispositivos-de-almacenamiento-usb)

---

## 🛡️ Punto A – GPO Credential Guard: habilitar seguridad basada en virtualización

### Parte 1 – Crear la Unidad Organizativa y la GPO
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. En el **Administrador del servidor**, ir a **Herramientas → Administración de directivas de grupo**.

2. En el árbol de la izquierda, hacer clic en el botón de expansión del bosque (`MiguelRM.local`) para desplegar los dominios.

3. Hacer clic derecho sobre el dominio `MiguelRM.local` → seleccionar **"Nueva unidad organizativa"**.

4. Ingresar el nombre `Empresa` y hacer clic en **Aceptar**.

5. Hacer clic derecho sobre la unidad organizativa `Empresa` recién creada → seleccionar **"Crear un GPO en este dominio y vincularlo aquí…"**.

6. Asignar el nombre `GPO Credential Guard` y hacer clic en **Aceptar**.

7. Hacer clic derecho sobre la GPO recién creada → seleccionar **Editar**.

### Parte 2 – Configurar la directiva de Credential Guard
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

8. En el Editor de administración de directivas de grupo, navegar hasta:
    ```
    Configuración del equipo
    └── Directivas
        └── Plantillas administrativas
            └── Sistema
                └── Device Guard
    ```

9. Hacer doble clic en **"Activar la seguridad basada en la virtualización"**.

10. En la ventana de configuración, seleccionar **Habilitada** y configurar las siguientes opciones:

    | Opción | Valor |
    |---|---|
    | Nivel de seguridad de la plataforma | Arranque seguro y protección de DMA |
    | Protección basada en virtualización de la integridad de código | **Habilitado con el bloqueo UEFI** |
    | Configuración de Credential Guard | **Habilitado con el bloqueo UEFI** |

11. Hacer clic en **Aplicar → Aceptar**.

### Parte 3 – Aplicar la GPO en el servidor
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

12. Abrir el **Símbolo del sistema (CMD)** como Administrador en el servidor y ejecutar:
    ```
    gpupdate /force
    ```
    Esperar a que se apliquen los cambios de directiva.

### Parte 4 – Verificar Credential Guard en el cliente (Windows 10)
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio)**

13. En la VM cliente con **Windows 10 Pro**, abrir **PowerShell** como Administrador.

14. Ejecutar el siguiente comando para verificar compatibilidad y estado de Credential Guard:
    ```powershell
    Get-CimInstance -ClassName win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard
    ```

15. En la salida del comando, verificar que `AvailableSecurityProperties` incluya los valores `{5, 6}`, lo que confirma que el hardware es compatible con Credential Guard y que la GPO fue desplegada correctamente al equipo del dominio.

> ✅ Credential Guard está habilitado y desplegado en los equipos del dominio `MiguelRM.local`.

> ⚠️ **Importante:** Si el comando devuelve el error `HRESULT 0x80041013 – Error en la carga del proveedor`, significa que la VM cliente **no tiene los prerrequisitos de hardware virtual habilitados**. Ver [Punto B](#punto-b--prerrequisitos-de-la-vm-cliente-para-credential-guard) para solucionarlo.

---

## ⚙️ Punto B – Prerrequisitos de la VM cliente para Credential Guard

> ℹ️ Este punto documenta la configuración necesaria en **VMware Workstation Pro 17** para que la VM cliente con Windows 10 Pro sea compatible con Credential Guard. Sin estos pasos, el comando de verificación del Punto A fallará.

Credential Guard requiere tres condiciones en la VM cliente:
- Firmware **UEFI** con Secure Boot habilitado
- **TPM 2.0** virtual
- **Virtualización anidada** (Nested Virtualization) habilitada

### Parte 1 – Apagar la VM y eliminar snapshots
> 🔧 **Ejecutar en: VMware Workstation (host físico)**

1. Apagar completamente la VM cliente de Windows 10 (no suspender).

2. En VMware, ir al menú **VM → Snapshots → Snapshot Manager**.

3. Eliminar todos los snapshots existentes.

> ⚠️ La encriptación de la VM no puede activarse mientras existan snapshots. Este paso es obligatorio.

### Parte 2 – Encriptar la VM
> 🔧 **Ejecutar en: VMware Workstation (host físico)**

4. Con la VM apagada, abrir **Virtual Machine Settings → Options → Access Control**.

5. Hacer clic en **Encrypt...** y asignar una contraseña segura.

6. Confirmar la operación. La VM quedará cifrada.

> ℹ️ El TPM virtual de VMware requiere que la VM esté encriptada para poder ser agregado.

### Parte 3 – Cambiar el firmware a UEFI y habilitar Secure Boot
> 🔧 **Ejecutar en: VMware Workstation (host físico)**

7. En **Virtual Machine Settings → Options → Advanced**, cambiar el tipo de firmware a **UEFI**.

8. Marcar la opción **"Enable secure boot"**.

9. Hacer clic en **Aceptar**.

### Parte 4 – Agregar el TPM virtual
> 🔧 **Ejecutar en: VMware Workstation (host físico)**

10. En **Virtual Machine Settings → Hardware**, hacer clic en **Add...**.

11. Seleccionar **Trusted Platform Module** y hacer clic en **Finish**.

> ✅ El módulo TPM aparecerá listado en el hardware de la VM.

### Parte 5 – Habilitar virtualización anidada
> 🔧 **Ejecutar en: VMware Workstation (host físico)**

12. En **Virtual Machine Settings → Hardware → Processors**, marcar:
    - ✅ **Virtualize Intel VT-x/EPT or AMD-V/RVI**
    - ✅ **Virtualize IOMMU (IO memory management unit)**

13. Hacer clic en **Aceptar** y encender la VM.

### Parte 6 – Volver a aplicar la GPO en el cliente
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio)**

14. En el cliente Windows 10, abrir **CMD** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```

15. Reiniciar el equipo cliente.

16. Luego del reinicio, verificar nuevamente con PowerShell como Administrador:
    ```powershell
    Get-CimInstance -ClassName win32_DeviceGuard -Namespace root\Microsoft\Windows\DeviceGuard
    ```

> ✅ Con los prerrequisitos correctamente configurados, el comando retornará las propiedades de Device Guard y `AvailableSecurityProperties` mostrará los valores `{5, 6}`.

---

## 🚫 Punto C – GPO Block USB: bloquear dispositivos de almacenamiento USB

### Parte 1 – Crear la GPO
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. En la consola de **Administración de directivas de grupo**, expandir el dominio y localizar la unidad organizativa `Empresa`.

2. Hacer clic derecho sobre `Empresa` → **"Crear un GPO en este dominio y vincularlo aquí…"**.

3. Asignar el nombre `GPO Block USB` y hacer clic en **Aceptar**.

4. Hacer clic derecho sobre la GPO recién creada → seleccionar **Editar**.

### Parte 2 – Configurar el bloqueo de USB
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

5. En el Editor de administración de directivas de grupo, navegar hasta:
    ```
    Configuración del equipo
    └── Directivas
        └── Plantillas administrativas
            └── Sistema
                └── Acceso de almacenamiento extraíble
    ```

6. Hacer doble clic en **"Todas las clases de almacenamiento extraíble: denegar acceso a todo"**.

7. Seleccionar **Habilitada** → hacer clic en **Aplicar → Aceptar**.

> ⚠️ **Nota:** Esta directiva tiene prioridad sobre cualquier otra configuración individual de almacenamiento extraíble. Una vez habilitada, ningún dispositivo USB, disco extraíble ni tarjeta de memoria podrá ser leído ni escrito en los equipos del dominio.

### Parte 3 – Aplicar la GPO
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

8. Abrir el **Símbolo del sistema (CMD)** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```
    Esperar a que se complete la actualización de directivas.

> ✅ La GPO de bloqueo USB está configurada y desplegada. Los equipos de la unidad organizativa `Empresa` dentro del dominio `MiguelRM.local` no podrán acceder a dispositivos de almacenamiento extraíble.

---

> **Nota:** Toda la práctica fue realizada en un entorno virtualizado con VMware Workstation Pro 17, utilizando Windows Server 2022 como controlador de dominio y Windows 10 Pro como cliente del dominio. La VM cliente requiere firmware UEFI, TPM 2.0 virtual, Secure Boot y virtualización anidada habilitados para que Credential Guard funcione correctamente.
