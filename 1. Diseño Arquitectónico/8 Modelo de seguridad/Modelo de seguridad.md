# Arquitectura de Seguridad del Sistema Híbrido CBT + ABT

```markdown
# Lista de Empaque

## Índice
* [Ropa](#ropa)
* [Calzado](#calzado)
* [Higiene](#higiene)
* [Electrónica](#electrónica)
* [Documentos](#documentos)

## <a id="ropa"></a>Ropa
*   **Artículos principales:** Camisetas, pantalones, vestidos, chaqueta
*   **Notas / Recordatorios:** Revisar clima del destino

## <a id="calzado"></a>Calzado
*   **Artículos principales:** Zapatillas, sandalias, zapatos formales
*   **Notas / Recordatorios:** Llevar calcetines suficientes

## <a id="higiene"></a>Higiene
*   **Artículos principales:** Cepillo de dientes, pasta, desodorante, shampoo
*   **Notas / Recordatorios:** Líquidos en envases pequeños

## <a id="electrónica"></a>Electrónica
*   **Artículos principales:** Teléfono, cargador, audífonos, adaptador
*   **Notas / Recordatorios:** Verificar voltaje del país

## <a id="documentos"></a>Documentos
*   *(Pendiente de completar)*
```

## Introducción
![WhatsApp Image 2025-11-27 at 11 17 43 AM](https://github.com/user-attachments/assets/7f8915f4-76f0-4a4f-b73e-7e89560da715)

La arquitectura de seguridad del sistema híbrido CBT + ABT se basa en un modelo multinivel, diseñado para proteger la integridad, confidencialidad y trazabilidad de las operaciones de identificación, validación y transacción. Este enfoque permite asegurar desde el medio de acceso del usuario hasta los servidores administrativos, integrando mecanismos robustos de autenticación, cifrado, validación cruzada y control de privilegios.

Este informe presenta una tabla resumen por niveles, una comparativa de tecnologías de identificación y una guía de recomendaciones de uso, con el fin de facilitar la comprensión, implementación y auditoría de los mecanismos de seguridad aplicados.

## Arquitectura Multinivel de Seguridad

| Nivel | Componentes principales | Reglas de seguridad aplicadas |
| :---: | :---------------------- | :---------------------------- |
| Nivel 0 | Tarjeta CBT, Cédula Azul, QR por Cuenta | Chips encriptados, Autenticación 2FA, Datos cifrados, Tokens |
| Nivel 1 | Validador con Lista Blanca | Validación equipo-usuario, Módulo SAM, Cifrado, Sello de tiempo, UUID, Hash token, Token REST |
| Nivel 2 | App Punto Recarga, App Movilízate, Web Usuario | Validación equipo-usuario, Token REST, Hash Token, Cifrado de datos |
| Nivel 3 | Server de Recursos, Web ABT, App CBT-ABT, Web Usuario | Control de acceso, Autenticación 2FA, Gestión de privilegios, Monitoreo de eventos, XSS |

## Comparativa de Tecnologías de Identificación

| Criterio de Seguridad | Tarjeta CIPURSE NFC | Cédula DNIe NFC | Imagen QR |
| :-------------------- | :------------------ | :-------------- | :---------- |
| Encriptación de datos | Alta – SAM cifrado | Alta – Chip cifrado | Alta – RSA cifrado |
| Prevención de duplicados | Alta – Sesión única | Alta – UID único | Alta – Step counter |
| Validación temporal | Alta – Sesión activa | Alta – Sesión activa | Alta – Expiración rápida |
| Validación de integridad | Alta – MAC | Alta – MAC | Alta – Hash y estructura |
| Autenticación | Alta – SAM mutua | Alta – MRZ segura | Alta – RSA privada |
| Ventana de duplicación | Alta – Sesión activa | Alta – Sesión activa | Alta – Timeout 500 ms |
| Datos sensibles protegidos | Alta – Acceso restringido | Alta – Acceso restringido | Alta – Encriptados |
| Resistencia a captura | Alta – Proximidad física | Alta – Proximidad física | Media-Alta – Expiración |
| Validación cruzada | Alta – SAM + sistema | Alta – Registro Civil | Alta – BalanceID + hash |
| Puntuación total | **9/9 – Alta** | **9/9 – Alta** | **8.5/9 – Alta** |

## Recomendaciones de Uso por Escenario

| Escenario | Método Recomendado | Justificación |
| :-------- | :----------------- | :------------ |
| Transacciones frecuentes de bajo monto | Tarjeta CIPURSE NFC o Cédula DNIe NFC | Rapidez y conveniencia son prioritarias |
| Transacciones de alto valor | Imagen QR | Mayor seguridad y trazabilidad |
| Entornos con alto riesgo de fraude | Imagen QR | Protección anti-replay y encriptación |
| Usuarios sin tarjeta CIPURSE NFC | Cédula DNIe NFC o Imagen QR | Alternativas disponibles para todos |
| Transacciones offline (sin conexión) | Imagen QR | Validación local completa sin servidor |

## Aplicaciones prácticas

*   **Fiscalización y auditoría**: permite verificar trazabilidad, autenticación y protección de datos en cada nivel.
*   **Diseño de interfaces**: guía la implementación de apps y validadores con seguridad integrada.
*   **Selección de tecnologías**: orienta la elección de medios de identificación según el contexto operativo.
*   **Documentación técnica**: base para manuales, APIs, matrices de requisitos y diagramas de arquitectura.


