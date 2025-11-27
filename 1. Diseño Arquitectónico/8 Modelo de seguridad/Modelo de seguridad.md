

[Index](#index)
* [Introducción a la Arquitectura de Seguridad CBT + ABT](#introducción-a-la-arquitectura-de-seguridad-cbt--abt)
* [Tabla resumen: Arquitectura de Seguridad CBT + ABT](#tabla-resumen-arquitectura-de-seguridad-cbt--abt)
* [Canales de comunicación](#canales-de-comunicación)

# Introducción a la Arquitectura de Seguridad CBT + ABT
![WhatsApp Image 2025-11-27 at 11 17 43 AM](https://github.com/user-attachments/assets/7f8915f4-76f0-4a4f-b73e-7e89560da715)
La presente arquitectura de seguridad define un modelo multinivel para proteger la integridad, confidencialidad y trazabilidad de las operaciones dentro del ecosistema híbrido CBT (Card-Based Ticketing) y ABT (Account-Based Ticketing). Esta estructura permite asegurar desde la identificación del usuario hasta la administración de privilegios y monitoreo de eventos, integrando mecanismos de autenticación, cifrado, validación y control de acceso.

Para facilitar su comprensión y aplicación, se presenta a continuación una tabla resumen por niveles, donde se detallan los componentes funcionales y las reglas de seguridad aplicadas en cada capa del sistema. Esta clasificación permite:

* Visualizar la progresión de seguridad desde el medio de pago hasta el backend administrativo.
* Asignar responsabilidades técnicas según el nivel de operación (campo, aplicación, servidor).
* Auditar y fiscalizar los mecanismos de protección implementados en cada interfaz y canal de comunicación.

La arquitectura está diseñada para garantizar una comunicación segura, tanto bidireccional como unidireccional, entre los distintos módulos del sistema, y para responder de forma robusta ante intentos de fraude, accesos no autorizados o inconsistencias operativas.

## Tabla resumen: Arquitectura de Seguridad CBT + ABT

| Nivel | Componentes principales | Reglas de seguridad aplicadas |
| :--- | :--- | :--- |
| Nivel 0 | - Tarjeta CBT<br>- Cédula Azul<br>- QR por Cuenta | - Chips encriptados<br>- Autenticación 2FA (CTA ABT)<br>- Datos cifrados<br>- Tokens |
| Nivel 1 | - Validador con Lista Blanca | - Validación equipo-usuario<br>- Módulo SAM<br>- Cifrado de datos<br>- Sello de tiempo<br>- UUID<br>- Hash token<br>- Token REST |
| Nivel 2 | - App Punto Recarga<br>- App Movilízate<br>- Web Usuario | - Validación equipo-usuario<br>- Token REST<br>- Hash Token<br>- Cifrado de datos |
| Nivel 3 | - Server de Recursos<br>- Web ABT<br>- App CBT-ABT<br>- Web Usuario | - Control de acceso<br>- Autenticación 2FA<br>- Gestión de privilegios<br>- Monitoreo de eventos<br>- Mecanismos XSS |

## Canales de comunicación

| Tipo | Descripción |
| :--- | :--- |
| **Bidireccional segura** | Comunicación entre apps, validadores y servidores con cifrado y autenticación |
| **Unidireccional segura** | Comunicación desde identificadores hacia validadores, protegida y controlada |

