<div align="center">

# 🔐 Práctica 1 – Seguridad de Sistemas Operativos

**Instituto Tecnológico de Las Américas (ITLA)**

| Campo | Detalle |
|---|---|
| **Materia** | Seguridad de Sistemas Operativos |
| **Instructor** | Juan Alexander Ramírez |
| **Estudiante** | Miguel Ramirez Meli |
| **Fecha** | 25-01-2026 |

</div>

---

## 📋 Tabla de Contenido

- [🖥️ Punto A – Instalar Windows Server 2022 en VMware](#punto-a--instalar-windows-server-2022-en-vmware)
- [🏛️ Punto B – Activar Active Directory y configurar el dominio](#punto-b--activar-active-directory-y-configurar-el-dominio)
- [💻 Punto C – Unir un cliente Windows 10 al dominio](#punto-c--unir-un-cliente-windows-10-al-dominio)

---

## 🖥️ Punto A – Instalar Windows Server 2022 en VMware

1. Abrir **VMware Workstation Pro** desde el escritorio.

2. En la pantalla de inicio, hacer clic en **"Create a New Virtual Machine"**.

3. En el asistente, dejar seleccionada la opción **"Typical (recommended)"** y hacer clic en **Siguiente**.

4. Seleccionar **"Installer disc image file (ISO)"** y hacer clic en **Browse** para buscar la imagen ISO de Windows Server.

5. Navegar hasta la carpeta donde se descargó la ISO (`SERVER_EVAL_x64FRE_es-es.iso`) y hacer clic en **Abrir**.

6. VMware detectará automáticamente la versión (**Windows Server 2022**) y activará el **Easy Install**. Hacer clic en **Siguiente**.

7. En la pantalla de Easy Install, hacer clic en **Siguiente** sin llenar la clave de producto. Cuando aparezca el aviso, seleccionar **Yes**.

8. Asignar un nombre a la máquina virtual (ej.: `MiguelRM`) y hacer clic en **Siguiente**.

9. Configurar el tamaño del disco (recomendado: **60 GB**) y seleccionar **"Split virtual disk into multiple files"** para mayor resiliencia. Hacer clic en **Siguiente**.

10. Revisar el resumen de hardware. Si se necesitan más recursos, hacer clic en **"Customize Hardware"**. Luego hacer clic en **Finish**.

11. Antes de encender la VM, ir a **"Edit virtual machine settings"**, seleccionar el dispositivo **Floppy** y hacer clic en **Remove**. Confirmar con **OK**.

> ⚠️ **Importante:** Este paso es necesario para evitar errores durante la instalación por el Easy Install.

12. Hacer clic en **"Power on this virtual machine"** para iniciar la instalación.

13. Cuando aparezca `"Press any key to boot from CD or DVD..."`, presionar **Enter** lo más rápido posible.

14. Seleccionar el idioma de instalación (Español) y hacer clic en **Siguiente**.

15. Hacer clic en **"Instalar ahora"**.

16. Seleccionar la edición del sistema operativo. Para interfaz gráfica, elegir la **2.ª opción** (`Windows Server 2022 Standard Evaluation – Experiencia de escritorio`). Hacer clic en **Siguiente**.

17. Aceptar los términos y condiciones. Hacer clic en **Siguiente**.

18. Seleccionar **"Personalizar: instalar solo el sistema operativo"**.

19. Seleccionar la partición disponible (60 GB sin asignar) y hacer clic en **Siguiente**.

20. Esperar a que finalice la instalación. El servidor se reiniciará automáticamente.

21. Establecer una contraseña para el Administrador y hacer clic en **Finalizar**.

22. Para iniciar sesión, usar la combinación: `Ctrl derecho + Alt derecho + Insert`.

23. Ingresar la contraseña configurada.

24. Una vez dentro, cambiar el nombre del equipo:
    - Ir a **Servidor local** en el Administrador del servidor.
    - Hacer clic en el nombre del equipo.
    - Seleccionar **Cambiar**.
    - Ingresar el nombre deseado y confirmar. Reiniciar cuando se solicite.

---

## 🏛️ Punto B – Activar Active Directory y configurar el dominio

1. En el **Administrador del servidor**, hacer clic en **"Agregar roles y características"**.

2. Hacer clic en **Siguiente** en la pantalla de bienvenida.

3. Dejar seleccionada la opción **"Instalación basada en características o en roles"**. Hacer clic en **Siguiente**.

4. Seleccionar el servidor de destino (aparece automáticamente). Hacer clic en **Siguiente**.

5. En la lista de roles, marcar **"Servicios de dominio de Active Directory"**.

6. Cuando aparezca el cuadro de dependencias, hacer clic en **"Agregar características"**.

7. Hacer clic en **Siguiente** hasta llegar a la pantalla de Confirmación.

8. Marcar la opción **"Reiniciar automáticamente el servidor de destino en caso necesario"**, confirmar con **Sí** y hacer clic en **Instalar**.

9. Esperar a que finalice la instalación. Hacer clic en **Cerrar**.

10. En la barra superior del Administrador del servidor, hacer clic en el ícono de **Notificaciones** (bandera con triángulo amarillo).

11. Hacer clic en **"Promover este servidor a controlador de dominio"**.

12. Seleccionar **"Agregar un nuevo bosque"** e ingresar el nombre de dominio raíz: `MiguelRM.local`. Hacer clic en **Siguiente**.

13. En las opciones del controlador de dominio:
    - Nivel funcional del bosque: **Windows Server 2016**
    - Nivel funcional del dominio: **Windows Server 2016**
    - Establecer la contraseña DSRM y hacer clic en **Siguiente**.

14. En la pantalla de opciones DNS, dejar desmarcada la opción de delegación DNS. Hacer clic en **Siguiente**.

15. El nombre NetBIOS se asigna automáticamente. Hacer clic en **Siguiente**.

16. Continuar haciendo clic en **Siguiente** hasta la pantalla de **Comprobación de requisitos previos**.

17. Si todas las comprobaciones son exitosas, hacer clic en **Instalar**.

18. El servidor se reiniciará automáticamente. Al volver, el dominio estará configurado.

19. Verificar en **Administrador del servidor → Servidor local** que aparezca el dominio `MiguelRM.local`.

---

## 💻 Punto C – Unir un cliente Windows 10 al dominio

1. En la VM con **Windows 10**, abrir el **Panel de Control** desde el menú inicio.

2. Ir a **Redes e Internet → Centro de redes y recursos compartidos**.

3. Hacer clic en **"Cambiar configuración del adaptador"**.

4. Hacer clic derecho sobre **Ethernet0** y seleccionar **Propiedades**.

5. Hacer doble clic en **"Protocolo de Internet versión 4 (TCP/IPv4)"**.

6. Seleccionar **"Usar las siguientes direcciones de servidor DNS"**:
   - **Servidor DNS preferido:** IP del servidor (verificar con `ipconfig` en el servidor, ej.: `192.168.0.10`)
   - **Servidor DNS alternativo:** `8.8.8.8`
   
   Hacer clic en **Aceptar**.

7. Abrir el **Explorador de archivos**, ir a **Este equipo**, hacer clic derecho en un espacio vacío y seleccionar **Propiedades**.

8. Hacer clic en **"Cambiar configuración"**.

9. En la ventana de **Propiedades del sistema**, hacer clic en **Cambiar**.

10. Configurar:
    - **Nombre del equipo:** `WS0829`
    - **Miembro del dominio:** marcar **Dominio** e ingresar `MiguelRM.local`
    
    Hacer clic en **Aceptar**.

11. Ingresar las credenciales del Administrador del dominio cuando se soliciten. Hacer clic en **Aceptar**.

12. Aparecerá el mensaje **"Se unió correctamente al dominio MiguelRM.local"**. Hacer clic en **Aceptar**.

13. Reiniciar el equipo cliente.

14. Al iniciar sesión, hacer clic en **"Otro usuario"** y verificar que aparezca la opción de iniciar sesión en el dominio `MIGUELRM`.

---

> **Nota:** Toda la práctica fue realizada utilizando VMware Workstation Pro 17 como hipervisor, con Windows Server 2022 Standard Evaluation y Windows 10 Pro como sistemas operativos invitados.
