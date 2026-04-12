[Practica4_Seguridad_SO (1).md](https://github.com/user-attachments/files/26659868/Practica4_Seguridad_SO.1.md)
<div align="center">

# 🔐 Práctica 4 – Seguridad de Sistemas Operativos

**Instituto Tecnológico de Las Américas (ITLA)**

| Campo | Detalle |
|---|---|
| **Materia** | Seguridad de Sistemas Operativos |
| **Instructor** | Juan Alexander Ramírez |
| **Autores** | Neury Montero Mejia & Miguel Ramirez Meli |
| **Fecha** | 22-02-2026 |

</div>

---

## 📋 Tabla de Contenido

- [🔏 Punto A – Habilitar DNSSEC en una zona de búsqueda del dominio](#punto-a--habilitar-dnssec-en-una-zona-de-búsqueda-del-dominio)
- [📡 Punto B – Propagar la configuración de DNSSEC a los clientes mediante GPO](#punto-b--propagar-la-configuración-de-dnssec-a-los-clientes-mediante-gpo)
- [✅ Comprobación – Verificar que DNSSEC fue aplicado correctamente](#comprobación--verificar-que-dnssec-fue-aplicado-correctamente)

---

## 🔏 Punto A – Habilitar DNSSEC en una zona de búsqueda del dominio

### Parte 1 – Abrir el Administrador de DNS y firmar la zona
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. En el **Administrador del servidor**, ir a **Herramientas → DNS**.

2. En el Administrador de DNS, expandir el servidor y luego **Forward Lookup Zones**. Hacer clic derecho sobre la zona `MiguelRM.local`.

3. Si la opción **DNSSEC** aparece en gris, seleccionar **Actualizar** en el mismo submenú y volver a intentarlo.

4. Hacer clic derecho sobre `MiguelRM.local` → **DNSSEC → Firmar la zona**.

5. Se desplegará el **Asistente para firmar zona**. Seleccionar **"Customize zone signing parameters"** y hacer clic en **Siguiente**.

6. En la pantalla **Key Master**, seleccionar **"The DNS server MIGUEL is the Key Master"** y hacer clic en **Siguiente**.

### Parte 2 – Configurar la Clave de Firma de Clave (KSK)
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

7. En la pantalla informativa de KSK, hacer clic en **Siguiente**.

8. En la pantalla **"Key Signing Key (KSK) – Configure one or more KSKs"**, hacer clic en **Add**.

9. En el formulario, configurar los siguientes valores:

    | Parámetro | Valor por defecto | Valor a usar |
    |---|---|---|
    | Algoritmo criptográfico | RSA/SHA-256 | RSA/SHA-256 (sin cambio) |
    | Longitud de clave (bits) | 2048 | 2048 (sin cambio) |
    | Frecuencia de sustitución (días) | 755 | **100** |

10. Cambiar la **Frecuencia de sustitución (días)** de `755` a `100`. Hacer clic en **Aceptar**.

11. Verificar que la KSK aparezca en la tabla y hacer clic en **Siguiente**.

### Parte 3 – Configurar la Clave de Firma de Zona (ZSK)
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

12. En la pantalla informativa de ZSK, hacer clic en **Siguiente**.

13. En la pantalla **"Zone Signing Key (ZSK) – Configure one or more ZSKs"**, hacer clic en **Add**.

14. En el formulario, configurar los siguientes valores:

    | Parámetro | Valor por defecto | Valor a usar |
    |---|---|---|
    | Longitud de clave (bits) | 1024 | **2048** |
    | Frecuencia de sustitución (días) | 90 | **100** |

15. Cambiar la **Longitud de clave** de `1024` a `2048` y la **Frecuencia de sustitución** de `90` a `100`. Hacer clic en **Aceptar**.

16. Verificar que la ZSK aparezca en la tabla y hacer clic en **Siguiente**.

### Parte 4 – Configurar NSEC3
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

17. En la pantalla **"Next Secure (NSEC)"**, seleccionar **"Use NSEC3"** (ya seleccionado por defecto) y dejar los valores por defecto:

    | Parámetro | Valor |
    |---|---|
    | Iterations | 50 |
    | Generate and use a random salt of length | 8 |

18. Hacer clic en **Next >**.

### Parte 5 – Configurar Trust Anchors
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

19. En la pantalla **"Trust Anchors (TAs)"**, marcar ambas casillas:
    - ✅ **Enable the distribution of trust anchors for this zone**
    - ✅ **Enable automatic update of trust anchors on key rollover (RFC 5011)**

20. Hacer clic en **Next >**.

### Parte 6 – Signing and Polling Parameters
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

21. En la pantalla **"Signing and Polling Parameters"**, dejar todos los valores por defecto:

    | Parámetro | Valor |
    |---|---|
    | DS record generation algorithm | SHA-1 and SHA-256 |
    | DS record TTL (seconds) | 3600 |
    | DNSKEY record TTL (seconds) | 3600 |
    | Secure delegation polling period (hours) | 12 |
    | Signature inception (hours) | 1 |

22. Hacer clic en **Next >**.

### Parte 7 – Finalizar la firma
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

23. Revisar el resumen y hacer clic en **Next >** para iniciar la firma.

24. Esperar a que el asistente complete el proceso. Cuando aparezca el mensaje **"La zona se firmó correctamente"**, hacer clic en **Finalizar**.

25. En el Administrador de DNS, hacer clic derecho sobre `MiguelRM.local` → **Actualizar**. Se visualizarán los nuevos registros DNSSEC (`RRSIG`, `DNSKEY`, `NSEC3`, `DS`) generados por la firma.

> ✅ La zona `MiguelRM.local` está firmada con DNSSEC. El estado cambiará de **"Not Signed"** a **"Signed"** en la lista de zonas.

---

## 📡 Punto B – Propagar la configuración de DNSSEC a los clientes mediante GPO

### Parte 1 – Crear la GPO DNSSEC
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. En el **Administrador del servidor**, ir a **Herramientas → Administración de directivas de grupo**.

2. En el árbol, expandir el dominio y localizar la unidad organizativa `Empresa`.

3. Hacer clic derecho sobre `Empresa` → **"Crear un GPO en este dominio y vincularlo aquí…"**.

4. Asignar el nombre `DNSSEC` y hacer clic en **Aceptar**.

5. Hacer clic derecho sobre la GPO recién creada → **Editar**.

### Parte 2 – Configurar la Directiva de Resolución de Nombres
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

6. En el Editor de administración de directivas de grupo, navegar hasta:
    ```
    Configuración del equipo
    └── Directivas
        └── Configuración de Windows
            └── Directiva de resolución de nombres
    ```

7. En el panel derecho, en la sección **"Crear reglas"**, configurar los siguientes campos:

    | Campo | Valor |
    |---|---|
    | ¿A qué parte del espacio de nombres se aplica esta regla? | **Sufijo** |
    | Nombre del sufijo | `MiguelRM.local` |

8. En el campo **"Entidad de certificación (Opcional)"**, hacer clic en **Examinar**.

9. En el recuadro de certificados que aparece, seleccionar el certificado disponible y hacer clic en **Aceptar**.

10. Marcar la casilla **"Habilitar DNSSEC en esta regla"**.

11. Bajo la sección **Configuración de DNSSEC → Validación**, marcar:
    - ✅ **Requerir a los clientes DNS que comprueben que el servidor DNS haya validado el nombre y la dirección**

12. Bajo la sección **IPsec**, marcar:
    - ✅ **Usar IPsec para la comunicación entre el cliente DNS y el servidor DNS**

13. Hacer clic en **Aplicar** en la parte inferior del panel.

### Parte 3 – Aplicar la GPO en el servidor
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

14. Abrir el **Símbolo del sistema (CMD)** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```
    Esperar a que se complete la actualización de directivas.

### Parte 4 – Aplicar la GPO en el cliente
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio)**

15. Abrir el **Símbolo del sistema (CMD)** como Administrador y ejecutar:
    ```
    gpupdate /force
    ```

16. Verificar que la directiva DNSSEC fue aplicada ejecutando:
    ```cmd
    gpresult /r
    ```
    Confirmar que `DNSSEC` aparece en la lista de **directivas de equipo aplicadas**.

> ✅ La GPO DNSSEC está configurada y desplegada. Los clientes del dominio dentro de la OU `Empresa` validarán las respuestas DNS mediante firmas DNSSEC y usarán IPsec para cifrar la comunicación con el servidor DNS.

---

> **Nota:** Toda la práctica fue realizada en un entorno virtualizado con VMware Workstation Pro 17, utilizando Windows Server 2022 como controlador de dominio y Windows 10 Pro como cliente del dominio. El equipo cliente debe estar dentro de la OU `Empresa` en Active Directory para que las GPOs sean aplicadas correctamente.

---

## ✅ Comprobación – Verificar que DNSSEC fue aplicado correctamente

### En el servidor – Verificar registros DNSSEC
> 🖥️ **Ejecutar en: Windows Server 2022 (Controlador de Dominio)**

1. Abrir el **Administrador de DNS** → **Forward Lookup Zones** → `MiguelRM.local`.

2. Verificar que existan los siguientes tipos de registros en la zona:

    | Registro | Descripción |
    |---|---|
    | `RRSIG` | Firma digital de los registros DNS |
    | `DNSKEY` | Claves públicas KSK y ZSK |
    | `NSEC3` | Prueba de no existencia autenticada |
    | `DS` | Delegación de firma hacia la zona padre |

3. Verificar que en la lista de zonas el estado de `MiguelRM.local` muestre **"Signed"** en la columna DNSSEC Status.

4. Desde **PowerShell** como Administrador, ejecutar:
    ```powershell
    Resolve-DnsName -Name MiguelRM.local -Type DNSKEY -Server 127.0.0.1
    ```
    Debe retornar los registros DNSKEY de la zona firmada.

### En el cliente – Verificar que la GPO DNSSEC fue aplicada
> 💻 **Ejecutar en: Windows 10 Pro (Cliente del dominio)**

5. Abrir **CMD** como Administrador y ejecutar:
    ```cmd
    gpresult /r
    ```
    Confirmar que `DNSSEC` aparece bajo **"Directivas de equipo aplicadas"**.

6. Verificar la resolución DNS con validación DNSSEC ejecutando en PowerShell:
    ```powershell
    Resolve-DnsName -Name MiguelRM.local -Type A -DnssecOk
    ```
    Si la respuesta incluye registros `RRSIG`, DNSSEC está funcionando correctamente en el cliente.

> ✅ DNSSEC está habilitado, firmado y propagado correctamente a los clientes del dominio `MiguelRM.local`.
