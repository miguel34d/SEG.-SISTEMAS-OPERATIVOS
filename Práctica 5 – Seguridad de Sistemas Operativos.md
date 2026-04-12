[Practica5_Seguridad_SO.md](https://github.com/user-attachments/files/26660045/Practica5_Seguridad_SO.md)
<div align="center">

# 🔐 Práctica 5 – Seguridad de Sistemas Operativos

**Instituto Tecnológico de Las Américas (ITLA)**

| Campo | Detalle |
|---|---|
| **Materia** | Seguridad de Sistemas Operativos |
| **Instructor** | Juan Alexander Ramírez |
| **Autores** | Neury Montero Mejia & Miguel Ramirez Meli |
| **Fecha** | 01-03-2026 |

</div>

---

## 📖 Conceptos Clave

| Concepto | Definición |
|---|---|
| **LAPS** | Herramienta de Microsoft que genera y rota automáticamente contraseñas únicas para el administrador local de cada equipo del dominio, almacenándolas en Active Directory. |
| **AdmPwd GPO Extension** | Componente de LAPS que se instala en los clientes para que puedan recibir y aplicar la política de gestión de contraseñas. |
| **Password Complexity** | Nivel de complejidad exigido a la contraseña: mayúsculas, minúsculas, números y caracteres especiales. |
| **Password Age** | Número de días de validez de la contraseña antes de que LAPS la rote automáticamente. |
| **Event ID** | Número único que identifica un tipo de evento registrado en el Visor de Eventos de Windows. |
| **Visor de Eventos** | Herramienta de Windows que registra actividad del sistema, seguridad y aplicaciones para auditoría y diagnóstico. |

---

## 📋 Tabla de Contenido

- [🔑 Punto A – Instalar LAPS para gestión de contraseñas de administrador local](#punto-a--instalar-laps-para-gestión-de-contraseñas-de-administrador-local)
- [⚙️ Punto B – Configurar LAPS mediante GPO](#punto-b--configurar-laps-mediante-gpo)
- [🔒 Punto C – Garantizar contraseñas con mínimo de 16 caracteres](#punto-c--garantizar-contraseñas-con-mínimo-de-16-caracteres)
- [📋 Punto D – Búsqueda y explicación de Event IDs en el Visor de Eventos](#punto-d--búsqueda-y-explicación-de-event-ids-en-el-visor-de-eventos)
- [✅ Comprobación – Verificar que LAPS está funcionando correctamente](#comprobación--verificar-que-laps-está-funcionando-correctamente)

---

## 🔑 Punto A – Instalar LAPS para gestión de contraseñas de administrador local
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

> **Prerequisito:** Descargar el archivo `LAPS.x64.msi` desde la página oficial de Microsoft antes de comenzar.

1. Hacer doble clic sobre el archivo `LAPS.x64` descargado en el escritorio. Se abrirá el asistente de instalación **"Local Administrator Password Solution Setup Wizard"**.

2. Hacer clic en **Next** hasta llegar a la pantalla de **"Custom Setup"**.

3. En la pantalla de Custom Setup, cada componente que muestre una **"X"** roja debe cambiarse a **"Will be installed on local hard drive"**. Hacer clic en el ícono de la X de cada uno y seleccionar esa opción. Los componentes a configurar son:

    | Componente | Acción |
    |---|---|
    | AdmPwd GPO Extension | Will be installed on local hard drive |
    | Management Tools | Will be installed on local hard drive |
    | Fat client UI | Will be installed on local hard drive |
    | PowerShell module | Will be installed on local hard drive |
    | GPO Editor templates | Will be installed on local hard drive |

4. Una vez todos los componentes muestren el ícono de disco (sin X roja), hacer clic en **Next**.

5. Hacer clic en **Next** nuevamente para confirmar y luego en **Install**. Al finalizar, hacer clic en **Finish**.

> ✅ LAPS quedó instalado con todos sus componentes en el disco local del servidor.

### Extender el esquema de Active Directory para LAPS
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

6. Abrir **PowerShell** como Administrador y ejecutar:
    ```powershell
    Import-Module AdmPwd.PS
    Update-AdmPwdADSchema
    ```

7. Otorgar permisos a los equipos de la OU `Empresa` para que puedan escribir su contraseña en AD:
    ```powershell
    Set-AdmPwdComputerSelfPermission -OrgUnit "Empresa"
    ```

> ✅ El esquema de Active Directory ahora incluye los atributos necesarios para LAPS (`ms-Mcs-AdmPwd` y `ms-Mcs-AdmPwdExpirationTime`).

---

## ⚙️ Punto B – Configurar LAPS mediante GPO
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. En el **Administrador del servidor**, ir a **Herramientas → Administración de directivas de grupo**.

2. En el árbol, expandir el dominio y localizar la unidad organizativa `Empresa`.

3. Hacer clic derecho sobre `Empresa` → **"Crear un GPO en este dominio y vincularlo aquí…"**.

4. Asignar el nombre `LAPS` y hacer clic en **Aceptar**.

5. Hacer clic derecho sobre la GPO recién creada → **Editar**.

6. En el Editor de administración de directivas de grupo, navegar hasta:
    ```
    Configuración del equipo
    └── Directivas
        └── Plantillas administrativas
            └── LAPS
    ```

7. Se mostrarán las siguientes opciones de configuración disponibles:

    | Directiva | Descripción corta |
    |---|---|
    | `Password Settings` | Define longitud, complejidad y vigencia de la contraseña |
    | `Name of administrator account to manage` | Especifica qué cuenta administrar (si no es la predeterminada) |
    | `Do not allow password expiration time longer than required by policy` | Fuerza el cambio al vencer sin permitir extensión |
    | `Enable local admin password management` | Activa la gestión de contraseña por LAPS en el equipo |

---

## 🔒 Punto C – Garantizar contraseñas con mínimo de 16 caracteres
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. Dentro de la carpeta **LAPS** del Editor de directivas, hacer doble clic en **"Password Settings"**.

2. Seleccionar **Habilitada** y configurar los siguientes parámetros:

    | Parámetro | Valor por defecto | Valor a usar |
    |---|---|---|
    | Password Complexity | Large + small + numbers + specials | Sin cambio |
    | Password Length | 14 | **16** |
    | Password Age (Days) | 30 | Sin cambio |

3. Cambiar **Password Length** de `14` a `16`. Hacer clic en **Aplicar → Aceptar**.

4. Hacer doble clic en **"Enable local admin password management"**.

5. Seleccionar **Habilitada** → hacer clic en **Aplicar → Aceptar**.

6. Hacer doble clic en **"Do not allow password expiration time longer than required by policy"**.

7. Seleccionar **Habilitada** → hacer clic en **Aplicar → Aceptar**.

> ℹ️ Esta directiva fuerza el cambio inmediato de la contraseña cuando llega su fecha de expiración, sin permitir que se extienda más allá del tiempo definido en la política.

8. Abrir el **Símbolo del sistema (CMD)** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```

> ✅ Las contraseñas de administrador local gestionadas por LAPS tendrán un mínimo de 16 caracteres con complejidad alta y se renovarán al vencer sin posibilidad de extensión.

### Instalar LAPS en el cliente
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio)**

9. Copiar el archivo `LAPS.x64.msi` al equipo cliente e instalarlo.

10. En la pantalla de **Custom Setup**, instalar únicamente:

    | Componente | Acción |
    |---|---|
    | AdmPwd GPO Extension | Will be installed on local hard drive |

11. Completar la instalación y ejecutar:
    ```
    gpupdate /force
    ```
12. Reiniciar el equipo cliente.

---

## 📋 Punto D – Búsqueda y explicación de Event IDs en el Visor de Eventos
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio) o Windows Server 2022**

### Cómo buscar un Event ID

1. Abrir el **Visor de Eventos**: presionar `Win + R`, escribir `eventvwr.msc` y dar **Enter**.
2. Expandir **Registros de Windows** en el panel izquierdo.
3. Seleccionar el registro correspondiente (**Sistema** o **Seguridad** según el evento).
4. En el panel derecho **Acciones**, hacer clic en **"Filtrar registro actual…"**.
5. En el campo de ID de evento, escribir el número deseado y hacer clic en **Aceptar**.

---

### Registros del Sistema

| Event ID | Nombre | Registro | Descripción |
|---|---|---|---|
| **1074** | System Shutdown / Restart | Sistema | Registra cuándo y por qué usuario se inició un apagado o reinicio del equipo. Útil para auditar cierres inesperados o no autorizados. |
| **6005** | Event Log Service Start | Sistema | Se genera cuando el servicio de Registro de Eventos arranca. Indica que el sistema fue encendido o reiniciado exitosamente. |
| **6006** | Event Log Service Stop | Sistema | Se genera cuando el servicio de Registro de Eventos se detiene de forma limpia. Indica un apagado ordenado del sistema. |

### Registros de Seguridad

> Para los eventos de seguridad, ir a **Registros de Windows → Seguridad** y filtrar por el ID correspondiente.

| Event ID | Nombre | Registro | Descripción |
|---|---|---|---|
| **4624** | Successful Logon | Seguridad | Auditoría de inicios de sesión exitosos. Registra el usuario, tipo de inicio de sesión y equipo de origen. Clave para monitorear accesos al sistema. |
| **4625** | Failed Logon | Seguridad | Auditoría de intentos fallidos de inicio de sesión. Muestra la cuenta usada, motivo del error y equipo de origen. Útil para detectar ataques de fuerza bruta. |
| **4719** | System Audit Policy Change | Seguridad | Se genera cuando un administrador modifica la directiva de auditoría del sistema. Si no hay registros, significa que no se han modificado las políticas de auditoría. |

---

## ✅ Comprobación – Verificar que LAPS está funcionando correctamente

### Verificar que la GPO fue aplicada al cliente
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio)**

1. Abrir **PowerShell** como Administrador y ejecutar:
    ```cmd
    gpresult /r /scope computer
    ```
    Confirmar que `LAPS` aparece bajo **"Objetos de directiva de grupo aplicados"**.

### Verificar la contraseña generada por LAPS
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

2. Abrir **PowerShell** como Administrador y ejecutar:
    ```powershell
    Get-AdmPwdPassword -ComputerName "NOMBRE_DEL_EQUIPO"
    ```
    Reemplazar `NOMBRE_DEL_EQUIPO` con el nombre del cliente (ej. `DESKTOP-ACPJ9HM`).

3. La salida debe mostrar:

    | Campo | Descripción |
    |---|---|
    | `ComputerName` | Nombre del equipo cliente |
    | `DistinguishedName` | Ruta en Active Directory |
    | `Password` | Contraseña generada por LAPS (16+ caracteres) |
    | `ExpirationTimestamp` | Fecha en que se rotará la contraseña |

> ✅ Si se muestra una contraseña de 16 o más caracteres, LAPS está funcionando correctamente y gestionando las credenciales del administrador local del equipo.

### Verificar longitud de la contraseña
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

4. Para confirmar que cumple el mínimo de 16 caracteres, ejecutar:
    ```powershell
    $pwd = (Get-AdmPwdPassword -ComputerName "DESKTOP-ACPJ9HM").Password
    $pwd.Length
    ```
    El resultado debe ser **16 o mayor**.

> ✅ LAPS está instalado, configurado mediante GPO y generando contraseñas únicas de 16 caracteres para el administrador local de los equipos en la OU `Empresa` del dominio `MiguelRM.local`.

---

> **Nota:** Toda la práctica fue realizada en un entorno virtualizado con VMware Workstation Pro 17, utilizando Windows Server 2022 como controlador de dominio y Windows 10 Pro como cliente del dominio. El equipo cliente debe estar dentro de la OU `Empresa` en Active Directory y tener instalado el componente `AdmPwd GPO Extension` de LAPS para que la gestión de contraseñas funcione correctamente.
