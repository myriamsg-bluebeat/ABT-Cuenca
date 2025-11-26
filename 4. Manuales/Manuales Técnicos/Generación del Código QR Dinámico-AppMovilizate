# Generación del Código QR Dinámico - App Movilízate
1. [🟩 Fase 1: Inicialización (`initQr`)](#-fase-1-inicialización-initqr)
   - [1. Carga de Datos](#1-carga-de-datos)
   - [2. Primera Generación](#2-primera-generación)

2. [🟦 Fase 2: Generación del QR (`generateQr`)](#-fase-2-generación-del-qr-generateqr)
   - [A. Encriptación de Datos Sensibles (Ocasional)](#a-encriptación-de-datos-sensibles-ocasional)
   - [B. Ofuscación de Identificadores (Cada Generación)](#b-ofuscación-de-identificadores-cada-generación)
   - [C. Ensamblaje Final del String del QR](#c-ensamblaje-final-del-string-del-qr)
     - [Estructura](#estructura)
     - [Ejemplo Práctico](#ejemplo-práctico)

3. [🛡️ Fase 3: Resumen de Seguridad](#-fase-3-resumen-de-seguridad)
   - [1. Seguridad por Tiempo (Ofuscación)](#1-seguridad-por-tiempo-ofuscación)
   - [2. Seguridad de Datos (Encriptación)](#2-seguridad-de-datos-encriptación)
   - [Ventajas del Sistema](#ventajas-del-sistema)

> El proceso para generar el código QR dinámico está diseñado para ser **seguro y eficiente**. Combina datos del usuario, un timestamp y una carga útil encriptada, todo ofuscado para dificultar su replicación. El flujo se divide en las siguientes fases:

---

## 🟩 Fase 1: Inicialización (`initQr`)

Este es el punto de partida la primera vez que el usuario abre la pantalla del QR.

### 1. Carga de Datos

Se llama a la función `createQr`, donde se pasa el `balanceId`, el cual nos permite crear un Qr atado a un balance, el cual nos retorna datos de la clave publica para encriptar los datos, el cual se almacena en el dispositivo de forma segura.

Se cargan desde el almacenamiento seguro los modelos de datos esenciales del usuario:

*   **Balance:** Contiene el saldo actual.
*   **DniModel:** Contiene la información de la tarjeta, incluyendo el `dniId` y la clave pública para la encriptación.

### 2. Primera Generación

Se invoca por primera vez al método `generateQr(init: true)`, lo que desencadena la creación del primer QR y la encriptación inicial de los datos.

---

## 🟦 Fase 2: Generación del QR (`generateQr`)

Este es el núcleo del proceso y se repite cada pocos segundos para que el QR sea siempre diferente. Se compone de tres subprocesos clave:

### A. Encriptación de Datos Sensibles (Ocasional)

Este subproceso se ejecuta al inicio (`init: true`) o cuando se fuerza una actualización (`generateEncrypt: true`). Su resultado se reutiliza en las generaciones posteriores hasta que se decida volver a encriptar.

*   **Método Invocado:** `encryptData`
*   **Datos a Encriptar:** Se combinan el `timestamp`, el `ID de usuario`, el `balance` y la `versión de la tarjeta`.
*   **Se usa sufijo:** Los datos se concatenan usando el sufijo `(#$#)`:
*   **Formato:**

```
(timestam/1)#$#timestamp#$#idUsuario#$#balance#$#version
```

*   **Cifrado:** Este string se encripta usando la **clave pública del usuario** (obtenida del `DniModel`), garantizando que solo el validador con la clave privada correspondiente pueda descifrarlo.
*   **Resultado:** Se obtiene un bloque de texto cifrado (`codeEncrypt`) que permanece constante durante varias generaciones del QR.

---

### B. Ofuscación de Identificadores (Cada Generación)

Este subproceso se ejecuta cada vez que se genera un nuevo QR para hacerlo único.

1.  **Permutación Aleatoria:** Se genera un nuevo orden aleatorio de los índices `"1"`, `"2"` y `"3"`.
    Ejemplo: `["3", "1", "2"]`. Esta permutación actúa como una "clave" para el orden de los datos.
2.  **Mapeo de Datos:**
    *   `"1"` → `idUsuario`
    *   `"2"` → `idBalance`
    *   `"3"` → `idDni`
3.  **Reordenamiento:** Usando la permutación aleatoria, los identificadores se colocan en un nuevo orden.
    Si la permutación fue `["3", "1", "2"]`, los datos se ordenan como `[idDni, idUsuario, idBalance]`.

---

### C. Ensamblaje Final del String del QR

Finalmente, todas las piezas se unen para formar el contenido final que se convertirá en la imagen del QR.

#### Estructura:

```
[perm1][timestamp][perm2][perm3][dato_ordenado1]$#$[dato_ordenado2]$#$[dato_ordenado3]$#$[datos_encriptados]
```

*   `[perm1]`, `[perm2]`, `[perm3]` → números de la permutación
*   `[timestamp]` → hora actual en segundos (epoch)
*   `[dato_ordenado_N]$#$` → identificadores reordenados, separados por el sufijo `$#$`
*   `[datos_encriptados]` → bloque `codeEncrypt` generado en el paso A.

#### Ejemplo Práctico

```
Permutación: ["2", "3", "1"]
Timestamp: 1678886400
Datos reordenados: [idBalance, idDni, idUsuario]
Suffix: $#$
CodeEncrypt: ENCRYPTED_DATA
Resultado final:
2167888640031idBalance$#$idDni$#$idUsuario$#$ENCRYPTED_DATA
```

---

## 🛡️ Fase 3: Resumen de Seguridad

Este mecanismo de doble capa proporciona una seguridad robusta:

### 1. Seguridad por Tiempo (Ofuscación):

El `timestamp` y la permutación aleatoria cambian cada pocos segundos, haciendo que cada QR sea único y de vida muy corta. Esto previene ataques de repetición (*replay attacks*) o el uso de capturas de pantalla.

### 2. Seguridad de Datos (Encriptación):

La información sensible (saldo, versión, etc.) está protegida con **criptografía asimétrica**. Incluso si alguien pudiera decodificar la estructura del QR, **no podría leer los datos más importantes sin la clave privada del validador**.

#### Ventajas del Sistema

```
QR de vida corta, imposible de reutilizar.
Cifrado robusto basado en criptografía asimétrica.
Estructura modular: cada fase puede probarse o actualizarse de forma independiente.
Compatible con validadores que operen offline o en tiempo real.
```
