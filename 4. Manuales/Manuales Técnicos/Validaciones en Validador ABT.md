```markdown
# Índice
* [Flujo de Validador Clipp ABT](#flujo-de-validador-clipp-abt)
  * [Tarjeta Movilízate](#tarjeta-movilízate)
  * [Pagos por QR](#pagos-por-qr)
  * [Cédula Azul (DNIe)](#cédula-azul-dnie)

---

# Flujo de Validador Clipp ABT
Diagrama funcional que muestra la interacción entre módulos externos (App SIR / SDK ABT) y módulos principales (App CLIPP / SAM–CARD).

## Tarjeta Movilízate

| App SIR (Externo) | App CLIPP | SAM / CARD | SDK ABT (Externo) |
| :--- | :--- | :--- | :--- |
| **1. Lectura Tarjeta** | **2. Genera Transacción** | **3. Lectura Bloque 46** | **4. CheckMovilizate** |
| Realiza proceso CBT; si saldo insuficiente, procede a ABT y envía ID de tarjeta a App CLIPP. | Crea una nueva transacción con su UUID y prepara lectura de bloque 46. | Lee versión, sincronización y saldo actual desde la tarjeta. | Valida información y puede devolver: **Correcto**, **Sin saldo** o **Incorrecto**. |
| | V | V | V |
| | **5. Respuesta SDK** | **6. Actualiza Bloque** | **7. PayMovilizate** |
| | Si **Incorrecto** → notifica FAIL a SIR. Si **Correcto** → comienza proceso de escritura. | Inicia proceso de escritura: SAM autoriza y la tarjeta guarda nuevos datos. Luego pasa a PayMovilizate. | Registra el pago y devuelve **OK / FAIL** a App CLIPP. |
| V | V | | |
| **8. Resultado de Transacción** | **8′. Finaliza** | | |
| Finaliza proceso y notifica resultado a SIR. | CLIPP cierra la operación y muestra resultado final. | | |

## Pagos por QR

| App SIR (Externo) | App CLIPP | SDK ABT (Externo) |
| :--- | :--- | :--- |
| **1. Escanea QR** | **2. Genera Transacción** | **3. CheckPayQR** |
| Recibe información del código QR, la procesa a texto plano y la envía a App CLIPP. | Crea transacción con UUID y envía data QR encriptada al SDK. | Desencripta, valida firma, tiempos y demás condiciones; devuelve **Correcto**, **Sin saldo** o **Incorrecto**. |
| V | V | |
| **4. Resultado Final** | **4′. Finalización de Transacción** | |
| Recibe respuesta de App CLIPP (OK / FAIL) y la muestra al usuario. | Termina proceso y envía resultado a App SIR. | |

## Cédula Azul (DNIe)

| App SIR (Externo) | App CLIPP | CARD DNIe | SDK ABT (Externo) |
| :--- | :--- | :--- | :--- |
| **1. Lectura UID** | **2. Nueva Transacción** | | **3. CheckDnie** |
| Detecta que no es Movilízate sino una Cédula Ecuatoriana; obtiene UID y lo envía a App CLIPP. | Crea transacción con UID y lo envía al SDK ABT (CheckDnie). | | Devuelve 3 tokens (N° Doc, Nacimiento, Expiración) a App CLIPP. |
| | | V | V |
| | | **4. Autenticación CHIP** | **5. PayDNIe** |
| | | En Card DNIe: utiliza los 3 tokens para autenticarse mediante protocolo BAC; si es correcto ejecuta PayDNIe. | Registra transacción y devuelve OK / FAIL a CLIPP. |
| V | V | | |
| **6. Resultado Final** | **6′. Finaliza Transacción** | | |
| SIR recibe resultado y muestra mensaje al usuario. | App CLIPP finaliza proceso y pasa resultado a App SIR. | | |

App SIR y SDK ABT son módulos externos. El flujo central (App CLIPP y SAM–CARD) representa la lógica principal del validador Clipp ABT.
```
