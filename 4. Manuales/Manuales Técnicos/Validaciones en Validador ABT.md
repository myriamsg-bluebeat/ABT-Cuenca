# Documentación Funcional del Validador Clipp ABT Híbrido

## Índice

1. [Flujo de Validación: Tarjeta Movilízate (Híbrido CBT/ABT)](#1-flujo-de-validación-tarjeta-movilízate-híbrido-cbtabt)
2. [Flujo de Pago: Códigos QR](#2-flujo-de-pago-códigos-qr)
3. [Flujo de Validación: Cédula Azul (DNIe)](#3-flujo-de-validación-cédula-azul-dnie)

---

Este documento describe los flujos funcionales principales del sistema de validación de transporte Clipp ABT (Account-Based Ticketing), el cual interactúa con componentes externos (App SIR, SDK ABT) para procesar pagos mediante diferentes medios: Tarjeta Movilízate (híbrido CBT/ABT), Códigos QR y Cédula Azul (DNIe).

---

## 1. 💳 Flujo de Validación: Tarjeta Movilízate (Híbrido CBT/ABT)

Este flujo prioriza la validación como tarjeta de saldo (CBT) y, si es insuficiente, intenta la validación basada en cuenta (ABT).

### Componentes Clave

| Módulo | Función |
| :--- | :--- |
| Tarjeta Movilízate | Medio físico de pago (Contiene saldo y datos de sincronización). |
| App CLIPP | Lógica central del Validador. Coordina la lectura, el débito y la respuesta. |
| SAM / CARD | Módulo de Seguridad de Acceso (SAM) o Módulo de Tarjeta. Autoriza operaciones criptográficas en el Validador. |
| SDK ABT | Módulo externo responsable de la lógica de cuentas ABT (saldos en la nube). |
| App SIR | Sistema de Información de Recaudo (Interfaz del operador/conductor). |

### Pasos del Proceso

| # | Módulo Emisor | Módulo Receptor | Acción / Descripción |
| :--- | :--- | :--- | :--- |
| 1 | Tarjeta Movilízate | App CLIPP | **Lectura Tarjeta:** Realiza el proceso CBT. Si el saldo es insuficiente en la tarjeta física, el sistema procede a ABT y envía el ID de la tarjeta a la App CLIPP para verificar saldo en la cuenta vinculada. |
| 2 | App CLIPP | SDK ABT | **Genera Transacción:** Crea una nueva transacción con un identificador único (UUID) y prepara la solicitud para leer datos específicos de la tarjeta. |
| 3 | Tarjeta Movilízate | App CLIPP | **Lectura Bloque 46:** Lee datos clave desde la tarjeta (versión, sincronización y saldo actual). |
| 4 | SDK ABT | App CLIPP | **CheckMovilizate:** Valida la información del paso 3 y la cuenta ABT vinculada. Devuelve: *Correcto*, *Sin saldo* o *Incorrecto*. |
| 5 | App CLIPP | App SIR | **Respuesta SDK:** Si el resultado es *Incorrecto* → notifica FAIL a App SIR. Si es *Correcto* → comienza el proceso de escritura del nuevo saldo en la tarjeta. |
| 6 | App CLIPP | Tarjeta Movilízate | **Actualiza Bloque:** Inicia la escritura del nuevo saldo. El SAM autoriza la operación y la tarjeta guarda los nuevos datos. Luego pasa a registrar el pago (*PayMovilizate*). |
| 7 | SDK ABT | App CLIPP | **PayMovilizate:** Registra el pago en el sistema central de cuentas (ABT) y devuelve OK / FAIL a App CLIPP. |
| 8 | App CLIPP | App SIR | **Resultado de Transacción:** Finaliza el proceso y notifica el resultado a App SIR para su visualización. |
| 8' | App CLIPP | | **Finaliza:** La App CLIPP cierra la operación y muestra el resultado final de la validación. |

## 2. 🤳 Flujo de Pago: Códigos QR

Este flujo permite el pago utilizando credenciales digitales generadas por una aplicación móvil, que son validadas por el SDK ABT.

### Componentes Clave

| Módulo | Función |
| :--- | :--- |
| App SIR | Interfaz de captura (cámara del validador). |
| App CLIPP | Lógica central del Validador. |
| SDK ABT | Módulo externo de validación de pagos digitales y saldos ABT. |

### Pasos del Proceso

| # | Módulo Emisor | Módulo Receptor | Acción / Descripción |
| :--- | :--- | :--- | :--- |
| 1 | App SIR | App CLIPP | **Escanea QR:** Recibe la información del código QR, la procesa a texto plano y la envía a la App CLIPP. |
| 2 | App CLIPP | SDK ABT | **Genera Transacción:** Crea la transacción con UUID y envía la data del QR encriptada al SDK ABT. |
| 3 | SDK ABT | App CLIPP | **CheckPayQR:** Desencripta la data, valida la firma, tiempos de expiración y demás condiciones de seguridad y saldo. Devuelve: *Correcto*, *Sin saldo* o *Incorrecto*. |
| 4 | App CLIPP | App SIR | **Resultado Final:** Recibe respuesta (OK / FAIL) y la muestra al usuario. |
| 4' | App CLIPP | App SIR | **Finalización de Transacción:** Termina el proceso y envía el resultado a App SIR. |

## 3. 🆔 Flujo de Validación: Cédula Azul (DNIe)

Este flujo utiliza la cédula de identidad electrónica (DNIe) como credencial de identificación para acceder a una cuenta de transporte ABT.

### Componentes Clave

| Módulo | Función |
| :--- | :--- |
| App SIR | Interfaz de captura de la cédula. |
| App CLIPP | Lógica central del Validador. |
| CARD DNIe | Chip de la cédula de identidad electrónica. |
| SDK ABT | Módulo externo de validación ABT y gestión de credenciales. |

### Pasos del Proceso

| # | Módulo Emisor | Módulo Receptor | Acción / Descripción |
| :--- | :--- | :--- | :--- |
| 1 | CARD DNIe | App CLIPP | **Lectura UID:** El validador detecta que es una Cédula Ecuatoriana (no Movilízate); obtiene el identificador único (UID) y lo envía a App CLIPP. |
| 2 | App CLIPP | SDK ABT | **Nueva Transacción:** Crea la transacción con el UID y lo envía al SDK ABT (*CheckDnie*). |
| 3 | SDK ABT | App CLIPP | **CheckDnie:** Valida el UID y devuelve tres tokens temporales (N° Doc, Nacimiento, Expiración) a App CLIPP. |
| 4 | App CLIPP | CARD DNIe | **Autenticación CHIP:** Utiliza los 3 tokens para autenticarse en el chip mediante el protocolo BAC (Basic Access Control). Si es correcto, el proceso ejecuta el pago (*PayDNIe*). |
| 5 | SDK ABT | App CLIPP | **PayDNIe:** Registra la transacción de pago en la cuenta ABT vinculada al DNIe y devuelve OK / FAIL a App CLIPP. |
| 6 | App CLIPP | App SIR | **Resultado Final:** SIR recibe el resultado y muestra el mensaje al usuario. |
| 6' | App CLIPP | App SIR | **Finaliza Transacción:** App CLIPP finaliza el proceso y pasa el resultado a App SIR. |
