# Auditoría de Configuración de Seguridad: Red WiFi Corporativa

> **Propósito:** Checklist de auditoría para identificar configuraciones predeterminadas, débiles o mal aplicadas en una red WiFi corporativa, junto con las mejores prácticas de endurecimiento (hardening) para cada hallazgo. Pensado para equipos de TI, seguridad de redes y administradores de infraestructura.

---

## 📌 Tabla de Contenidos

1. [Por qué la configuración WiFi es un punto crítico](#por-qué-la-configuración-wifi-es-un-punto-crítico)
2. [Áreas de la auditoría](#áreas-de-la-auditoría)
3. [Autenticación y cifrado](#1-autenticación-y-cifrado)
4. [Credenciales y configuración del controlador/AP](#2-credenciales-y-configuración-del-controladorap)
5. [Segmentación de red](#3-segmentación-de-red)
6. [Gestión de dispositivos y acceso físico](#4-gestión-de-dispositivos-y-acceso-físico)
7. [Monitoreo y detección](#5-monitoreo-y-detección)
8. [Configuraciones predeterminadas más comunes a corregir](#configuraciones-predeterminadas-más-comunes-a-corregir)
9. [Plan de endurecimiento priorizado](#plan-de-endurecimiento-priorizado)
10. [Checklist de auditoría completo](#checklist-de-auditoría-completo)

---

![Auditoría de seguridad: red WiFi corporativa]<img width="2800" height="3648" alt="infografia-wifi-audit" src="https://github.com/user-attachments/assets/32128117-b15f-46fc-bc50-0fd39f24a385" />


## Por qué la configuración WiFi es un punto crítico

La red WiFi corporativa suele ser el punto de entrada con menor escrutinio frente a otros controles de seguridad, porque una vez configurada, rara vez se revisa de nuevo. Un atacante con alcance de señal (desde el estacionamiento, un edificio contiguo, o incluso desde la calle) no necesita acceso físico a las instalaciones para intentar comprometerla. Una configuración débil aquí puede traducirse directamente en acceso a la red interna, sin pasar por firewall perimetral ni controles de acceso físico.

---

## Áreas de la auditoría

```
Autenticación y cifrado → Credenciales del AP → Segmentación → Gestión de dispositivos → Monitoreo
```

Cada área se audita de forma independiente, pero los hallazgos suelen estar relacionados: una red sin segmentación agrava el impacto de una credencial débil, y la falta de monitoreo retrasa la detección de cualquier abuso de las anteriores.

---

## 1. Autenticación y cifrado

**Qué revisar:**

- **Protocolo de seguridad en uso.** WEP y WPA (versión 1) están obsoletos y son trivialmente vulnerables; WPA2 sin WPA3 sigue siendo aceptable pero ya no es el estándar recomendado para entornos corporativos nuevos.
- **Modo de autenticación.** WPA2/WPA3-Personal (PSK, una sola contraseña compartida) en lugar de WPA2/WPA3-Enterprise (802.1X con servidor RADIUS y credenciales individuales) es una señal de alerta en entorno corporativo, salvo en redes de invitados aisladas.
- **Fortaleza de la passphrase**, si se usa modo Personal en alguna red (ej. WiFi de invitados): longitud, complejidad y si ha sido rotada alguna vez desde la instalación.
- **Protocolos de cifrado heredados habilitados** por compatibilidad retroactiva (TKIP en vez de AES/CCMP), que debilitan la red completa aunque la mayoría de dispositivos usen el protocolo más fuerte.

**Buenas prácticas de endurecimiento:**

- Migrar a **WPA3-Enterprise** donde el hardware lo soporte; como mínimo, **WPA2-Enterprise con AES/CCMP**, nunca TKIP.
- Implementar **802.1X con servidor RADIUS** para autenticación individual por usuario o dispositivo, no una contraseña compartida para toda la organización.
- Si existe una red con PSK (por ejemplo, IoT o invitados), usar una passphrase larga (20+ caracteres, generada aleatoriamente) y rotarla periódicamente.
- Deshabilitar explícitamente WEP, WPA1 y TKIP en la configuración del controlador, incluso si "nadie los usa" — su sola disponibilidad es una superficie de ataque.

---

## 2. Credenciales y configuración del controlador/AP

**Qué revisar:**

- **Credenciales administrativas predeterminadas** del controlador WiFi o puntos de acceso (admin/admin, admin/password, o las que trae de fábrica el fabricante) sin cambiar.
- **SSID predeterminado del fabricante** sin modificar, que revela marca y modelo del equipo a cualquier atacante cercano, facilitando la búsqueda de vulnerabilidades conocidas para ese hardware específico.
- **Acceso de administración expuesto** vía la propia red WiFi o, peor, accesible desde internet sin restricción.
- **Firmware desactualizado** en controladores y puntos de acceso, sin proceso definido de actualización periódica.
- **Servicios de gestión remota innecesarios habilitados** (Telnet, HTTP sin cifrar, SNMP con community string por defecto "public").

**Buenas prácticas de endurecimiento:**

- Cambiar todas las credenciales administrativas por defecto inmediatamente tras la instalación, usando contraseñas únicas y robustas, idealmente gestionadas por un gestor de contraseñas corporativo.
- Renombrar el SSID a algo que no revele marca, modelo o información organizacional sensible (evitar nombres como "Oficina-Gerencia" o "ATTwifi123").
- Restringir el acceso a la interfaz de administración a una VLAN de gestión dedicada, accesible solo desde la red cableada interna o mediante VPN — nunca desde la WiFi general ni desde internet.
- Establecer un calendario de actualización de firmware y suscribirse a los avisos de seguridad del fabricante.
- Deshabilitar Telnet, HTTP (usar HTTPS) y SNMP v1/v2 (usar SNMPv3 si es necesario), y cambiar cualquier community string por defecto.

---

## 3. Segmentación de red

**Qué revisar:**

- **Ausencia de VLANs**, con todos los dispositivos (laptops corporativos, impresoras, cámaras IP, dispositivos IoT, invitados) en la misma red plana.
- **Red de invitados con acceso a recursos internos**, sin aislamiento real del resto de la infraestructura.
- **Dispositivos IoT y OT en la misma VLAN que equipos de usuario final**, a pesar de tener perfiles de seguridad y parches muy distintos.
- **Reglas de firewall inter-VLAN inexistentes o demasiado permisivas** que anulan el propósito de la segmentación.

**Buenas prácticas de endurecimiento:**

- Implementar **VLANs separadas** como mínimo para: red corporativa de usuarios, red de invitados, dispositivos IoT/OT, y red de gestión de infraestructura.
- Asegurar que la **red de invitados esté completamente aislada**, sin visibilidad hacia la red interna, con su propio acceso a internet y límites de ancho de banda.
- Aplicar **reglas de firewall explícitas entre VLANs**, permitiendo solo el tráfico estrictamente necesario (lista blanca, no lista negra).
- Considerar **microsegmentación o modelo zero trust** para entornos con alta sensibilidad de datos, donde incluso dentro de una misma VLAN se valida cada conexión.

---

## 4. Gestión de dispositivos y acceso físico

**Qué revisar:**

- **Ausencia de control de acceso a la red (NAC)** que verifique el cumplimiento del dispositivo (antivirus activo, parches al día) antes de otorgar acceso.
- **Puntos de acceso físicamente expuestos** o accesibles sin supervisión, vulnerables a manipulación física o sustitución por un dispositivo rogue.
- **Falta de inventario actualizado** de puntos de acceso autorizados, lo que dificulta detectar un AP no autorizado (rogue AP) instalado por error o con intención maliciosa.
- **Dispositivos BYOD sin política clara** de qué nivel de acceso se les otorga frente a equipos corporativos gestionados.

**Buenas prácticas de endurecimiento:**

- Implementar **NAC (Network Access Control)** para validar el estado de salud del dispositivo antes de conceder acceso pleno a la red.
- Asegurar físicamente los puntos de acceso (montaje en altura, gabinetes con llave en zonas públicas) y habilitar alertas de manipulación si el hardware lo soporta.
- Mantener un **inventario actualizado de APs autorizados** (ubicación, dirección MAC, modelo) y ejecutar barridos periódicos para detectar APs no autorizados (rogue AP detection), muchas soluciones WLAN empresariales lo incluyen de forma nativa.
- Definir una **política BYOD explícita**, idealmente con acceso restringido a una VLAN separada y sin alcance a recursos internos sensibles.

---

## 5. Monitoreo y detección

**Qué revisar:**

- **Ausencia de logging centralizado** de eventos de autenticación WiFi (conexiones exitosas, fallidas, desconexiones).
- **Sin alertas configuradas** para patrones anómalos: múltiples intentos fallidos de autenticación, conexión de un dispositivo desde una ubicación geográfica inconsistente, o aparición de un nuevo SSID similar al corporativo (posible ataque de gemelo malvado / evil twin).
- **Sin capacidad de detección de APs rogue o ataques de deautenticación** activos en el entorno.
- **Revisión de logs no programada**, dependiendo únicamente de la reacción ante un incidente ya ocurrido.

**Buenas prácticas de endurecimiento:**

- Centralizar los logs del controlador WiFi en el **SIEM corporativo**, con reglas de correlación específicas para autenticación inalámbrica.
- Configurar alertas automáticas para: ráfagas de fallos de autenticación, SSIDs duplicados o similares al corporativo, y conexión de dispositivos no reconocidos por el NAC.
- Habilitar **detección de intrusión inalámbrica (WIDS/WIPS)** si la plataforma lo soporta, para identificar ataques de deautenticación, rogue APs y intentos de spoofing.
- Establecer una **revisión periódica programada** (mensual o trimestral) de la configuración completa de la red WiFi, no solo reaccionar ante incidentes.

---

## Configuraciones predeterminadas más comunes a corregir

| Configuración por defecto | Riesgo | Acción de hardening |
|---|---|---|
| Usuario/contraseña admin de fábrica | Acceso administrativo trivial | Cambiar inmediatamente por credenciales únicas y robustas |
| SSID con marca/modelo del equipo | Facilita identificar vulnerabilidades del hardware | Renombrar a algo neutral, sin información organizacional |
| WPS habilitado | Bypass de la autenticación WPA mediante PIN vulnerable | Deshabilitar WPS en todos los puntos de acceso |
| TKIP habilitado junto a AES | Debilita el cifrado de toda la red | Forzar solo AES/CCMP |
| SNMP con community string "public" | Lectura/escritura no autorizada de configuración | Migrar a SNMPv3 o deshabilitar si no se usa |
| Gestión remota accesible desde WAN | Exposición del panel de control a internet | Restringir a VLAN de gestión interna o VPN |
| Sin VLAN para invitados | Acceso de visitantes a la red interna | Segmentar con VLAN dedicada y aislada |
| Firmware sin actualizar desde la instalación | Vulnerabilidades conocidas sin parchear | Calendario de actualización y suscripción a avisos del fabricante |

---

## Plan de endurecimiento priorizado

Si los recursos son limitados, este es un orden recomendado de implementación según impacto/esfuerzo:

1. **Cambiar todas las credenciales por defecto** (administrativas y SNMP) — impacto alto, esfuerzo mínimo.
2. **Deshabilitar WPS, WEP, WPA1 y TKIP** — impacto alto, esfuerzo mínimo.
3. **Migrar a WPA2/WPA3-Enterprise con 802.1X** — impacto alto, esfuerzo medio-alto (requiere servidor RADIUS).
4. **Segmentar con VLANs** (invitados, IoT, corporativo, gestión) — impacto alto, esfuerzo medio.
5. **Restringir la gestión del controlador a una VLAN dedicada** — impacto medio-alto, esfuerzo bajo.
6. **Actualizar firmware y establecer calendario de parches** — impacto medio, esfuerzo bajo-medio.
7. **Implementar NAC** — impacto alto, esfuerzo alto (proyecto de mayor envergadura).
8. **Habilitar WIDS/WIPS y centralizar logs en el SIEM** — impacto medio-alto, esfuerzo medio.

---

## Checklist de auditoría completo

- [ ] Protocolo de seguridad confirmado como WPA2/WPA3, sin WEP ni WPA1 habilitados
- [ ] TKIP deshabilitado, solo AES/CCMP en uso
- [ ] WPS deshabilitado en todos los puntos de acceso
- [ ] Red corporativa en modo Enterprise (802.1X/RADIUS), no PSK compartido
- [ ] Credenciales administrativas del controlador/AP cambiadas de fábrica
- [ ] SSID sin información de marca, modelo u organización sensible
- [ ] Acceso de gestión restringido a VLAN dedicada o VPN
- [ ] Servicios innecesarios deshabilitados (Telnet, HTTP, SNMP v1/v2)
- [ ] Firmware actualizado y con calendario de mantenimiento definido
- [ ] VLANs separadas para corporativo, invitados, IoT/OT y gestión
- [ ] Red de invitados completamente aislada de la red interna
- [ ] Reglas de firewall inter-VLAN configuradas como lista blanca
- [ ] Inventario actualizado de puntos de acceso autorizados
- [ ] NAC u otro control de cumplimiento de dispositivo implementado
- [ ] Logs de autenticación WiFi centralizados en el SIEM
- [ ] Alertas configuradas para fallos de autenticación y SSIDs duplicados
- [ ] WIDS/WIPS habilitado si la plataforma lo soporta
- [ ] Revisión periódica de la configuración programada (no solo reactiva)

---

## ⚠️ Aviso

Este documento es una guía de auditoría a nivel de configuración y buenas prácticas, pensada para uso defensivo por parte de equipos de TI y seguridad sobre infraestructura propia o bajo autorización explícita. No incluye procedimientos de explotación de redes inalámbricas. Cualquier prueba activa de seguridad (pentesting inalámbrico) debe realizarse únicamente con autorización formal y dentro del alcance acordado.

---

*Última actualización: junio 2026*
