<div align="center">

# 🔐 Práctica 2 – Seguridad de Sistemas Operativos

**Instituto Tecnológico de Las Américas (ITLA)**

| Campo | Detalle |
|---|---|
| **Materia** | Seguridad de Sistemas Operativos |
| **Instructor** | Juan Alexander Ramírez |
| **Estudiante** | Miguel Ramirez Meli |
| **Fecha** | 01-02-2026 |

</div>

---

## 📋 Tabla de Contenido

- [🔒 Punto A – GPO BitLocker: cifrado de disco con PIN y respaldo en Active Directory](#punto-a--gpo-bitlocker-cifrado-de-disco-con-pin-y-respaldo-en-active-directory)
- [🚫 Punto B – GPO NTLM: bloquear autenticación NTLM en el dominio](#punto-b--gpo-ntlm-bloquear-autenticación-ntlm-en-el-dominio)

---

## 🔒 Punto A – GPO BitLocker: cifrado de disco con PIN y respaldo en Active Directory

### Parte 1 – Instalar la característica BitLocker

1. En el **Administrador del servidor**, hacer clic en **Administrar → Agregar roles y características**.

2. Hacer clic en **Siguiente** hasta llegar a la pantalla **Características**. Marcar **"Cifrado de unidad BitLocker"**.

3. Cuando aparezca el cuadro de dependencias, hacer clic en **"Agregar características"**.

4. Hacer clic en **Siguiente** hasta llegar a la pantalla de Confirmación.

5. Marcar **"Reiniciar automáticamente el servidor de destino en caso necesario"**, confirmar con **Sí** y hacer clic en **Instalar**.

6. Esperar a que finalice la instalación.

### Parte 2 – Crear y configurar la GPO de BitLocker

7. Ir a **Herramientas → Administración de directivas de grupo**.

8. Hacer clic derecho sobre el dominio (`MiguelRM.local`) y seleccionar **"Crear un GPO en este dominio y vincularlo aquí"**.

9. Asignar el nombre `GPO BitLocker` y hacer clic en **Aceptar**.

10. Expandir el dominio en el árbol, hacer clic derecho sobre la GPO recién creada y seleccionar **Editar**.

11. Navegar por el árbol hasta:
    ```
    Configuración del equipo
    └── Directivas
        └── Plantillas administrativas
            └── Componentes de Windows
                └── Cifrado de unidad BitLocker
                    └── Unidades del sistema operativo
    ```

12. Hacer doble clic en **"Requerir autenticación adicional al iniciar"** → seleccionar **Habilitada** → **Aplicar → Aceptar**.

13. Hacer doble clic en **"Permitir los PIN mejorados para el inicio"** → seleccionar **Habilitada** → **Aplicar → Aceptar**.

14. Hacer doble clic en **"Configurar longitud mínima de PIN para el inicio"** → seleccionar **Habilitada** → establecer mínimo de caracteres: **4** → **Aplicar → Aceptar**.

15. Hacer doble clic en **"Configurar el perfil de validación de plataforma del TPM"** → seleccionar **Habilitada** → **Aplicar → Aceptar**.

16. En la sección raíz de **Cifrado de unidad BitLocker**, habilitar también:
    **"Almacenar información de recuperación de BitLocker en los Servicios de dominio de Active Directory"** → **Habilitada** → **Aplicar → Aceptar**.

### Parte 3 – Aplicar la GPO y cifrar el disco

17. Abrir el **Símbolo del sistema (CMD)** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```
    Esperar a que se apliquen los cambios.

18. En el **Explorador de archivos**, crear una carpeta en el escritorio llamada `Key recovery` para guardar la clave de recuperación.

19. Ir a **Este equipo**, abrir **Administración de discos** (clic derecho en el disco C → **Reducir volumen**), establecer el tamaño deseado para la nueva partición.

20. En el espacio no asignado resultante, hacer clic derecho → **"Nuevo volumen simple"** y seguir el asistente.

21. Una vez creada la nueva partición, hacer clic derecho sobre ella en el Explorador de archivos → **"Activar BitLocker"**.

22. Elegir el método de desbloqueo: seleccionar **"Usar una contraseña para desbloquear la unidad"**, ingresar la contraseña deseada y hacer clic en **Siguiente**.

23. Elegir dónde guardar la clave de recuperación → **"Guardar en un archivo"** → navegar hasta la carpeta `Key recovery` → **Guardar → Siguiente**.

24. Seleccionar **"Cifrar la unidad entera"** → **Siguiente**.

25. Elegir el modo de cifrado: **"Modo Compatible"** (recomendado para unidades extraíbles) → **Siguiente**.

26. Hacer clic en **"Iniciar cifrado"** y esperar a que finalice.

> ✅ El cifrado de la unidad se ha completado. El ícono del candado aparecerá sobre la unidad en el Explorador de archivos.

---

## 🚫 Punto B – GPO NTLM: bloquear autenticación NTLM en el dominio

### Parte 1 – Crear la GPO NTLM

1. En el **Administrador del servidor**, hacer clic en **Herramientas → Administración de directivas de grupo**.

2. Hacer clic derecho sobre el dominio (`MiguelRM.local`) → **"Crear un GPO en este dominio y vincularlo aquí"**.

3. Asignar el nombre `GPO NTLM` y hacer clic en **Aceptar**.

4. Hacer clic derecho sobre la GPO recién creada → **Editar**.

### Parte 2 – Configurar las directivas NTLM

5. Navegar por el árbol hasta:
    ```
    Configuración del equipo
    └── Directivas
        └── Configuración de Windows
            └── Configuración de seguridad
                └── Directivas locales
                    └── Opciones de seguridad
    ```

6. Buscar la directiva **"Seguridad de red: restringir NTLM: auditar la autenticación NTLM en este dominio"**.

7. Hacer doble clic → marcar **"Definir esta configuración de directiva"** → cambiar el valor a **"Habilitar todo"** → **Aplicar → Aceptar**.

8. Buscar la directiva **"Seguridad de red: restringir NTLM: autenticación NTLM en este dominio"**.

9. Hacer doble clic → marcar **"Definir esta configuración de directiva"** → cambiar el valor a **"Denegar todo"** → **Aplicar → Sí → Aceptar**.

> ⚠️ **Nota:** La opción "Denegar todo" bloquea todos los intentos de autenticación NTLM en el dominio, previniendo accesos no autorizados a través del tráfico de red.

### Parte 3 – Aplicar la GPO

10. Abrir el **Símbolo del sistema (CMD)** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```
    Esperar a que se complete la actualización.

11. En la consola de **Administración de directivas de grupo**, hacer clic derecho sobre **GPO NTLM** → **Actualizar** para confirmar que la directiva está habilitada y desplegada.

> ✅ La GPO NTLM está configurada y desplegada en todos los equipos del dominio `MiguelRM.local`.

---

> **Nota:** Toda la práctica fue realizada en un entorno virtualizado con VMware Workstation Pro 17, utilizando Windows Server 2022 como controlador de dominio y Windows 10 Pro como cliente del dominio.
