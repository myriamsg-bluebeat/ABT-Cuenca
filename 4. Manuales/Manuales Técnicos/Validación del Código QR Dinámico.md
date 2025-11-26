# Documentación Técnica: Validación de QR - Validador

## 📋 Índice
1. [🔄 Flujo de Transacción Completo (Método QR)](#-flujo-de-transacción-completo-método-qr)
2. [🛠️ Procesamiento Local](#️-procesamiento-local)
   - [1. checkQR (`suspend fun`)](#1-checkqr-suspend-fun)
   - [2. tariffEvaluator (`private suspend fun`)](#2-tariffevaluator-private-suspend-fun)
   - [3. insertBDTransaction (`private suspend fun`)](#3-insertbdtransaction-private-suspend-fun)
   - [4. registerPayment (`suspend fun`)](#4-registerpayment-suspend-fun)
   - [5. saveNewBalance (`private suspend fun`)](#5-savenewbalance-private-suspend-fun)
3. [📡 Sincronización (RabbitMQ)](#-sincronización-rabbitmq)
   - [1. setupConsumer (`private fun`)](#1-setupconsumer-private-fun)

---

> Este documento detalla el flujo de validación y las funciones internas del dispositivo **Validador ABT (Sistema de Transacciones)**, enfocándose exclusivamente en el **procesamiento de códigos QR**.  
> El proceso asegura la autenticidad, vigencia y correcta aplicación de tarifas.  
> **Proyecto:** ABT - Sistema de Transacciones | **Fecha:** Octubre 2025

---

## 🔄 Flujo de Transacción Completo (Método QR)

El proceso de pago para el Código QR sigue una secuencia clara de validación, procesamiento y sincronización.

### Secuencia de Ejecución de Funciones
1. Usuario muestra imagen QR (pantalla o impresa).
2. **checkQR()** → Valida, decodifica y desencripta QR.
3. **tariffEvaluator()** → Verifica saldo, compara versiones y calcula el nuevo balance.
4. **registerPayment()** → Confirma el pago e inicia la sincronización.
5. **setupConsumer()** → Inicia la sincronización con otros dispositivos vía RabbitMQ.

**Seguridad:** Alta (RSA + anti-replay)  
**Datos necesarios:** Imagen QR (pantalla/impresa)

---

## 🛠️ Procesamiento Local

### 1. checkQR (`suspend fun`)
**Archivo:** `HomeController.kt` | **Tipo:** PÚBLICA SUSPEND  

Valida y procesa un código QR para realizar una transacción. Decodifica la imagen QR, verifica su autenticidad mediante desencriptación, valida su vigencia temporal y procesa el pago correspondiente.  
**Llama a `tariffEvaluator()` para la fase de cobro.**

#### Parámetros y Retorno
| Parámetro       | Tipo   | Descripción |
|-----------------|--------|-------------|
| `transactionId` | String | Identificador único de la transacción |
| `qr`            | String | Código QR en formato string con información encriptada |

**Retorno:** `JSONObject` - Resultado de la operación (éxito o error con detalles)

#### Fases de Procesamiento Interno
1. ✅ Validaciones Básicas del QR (timeout 500ms)  
2. 🔐 Decodificación y Extracción de Datos (Divide QR en **4 secciones**)  
3. 💾 Consulta en Base de Datos (Verifica **clave privada**)  
4. 🔓 Desencriptación y Validación (**Sección 4 con RSA**)  
5. 🛡️ Validaciones de Seguridad (**expiración temporal**, prevención de **replay attacks con steep**)  
6. 💳 Procesamiento de la Transacción (llama a `tariffEvaluator()`)

---

### 2. tariffEvaluator (`private suspend fun`)
**Archivo:** `HomeController.kt` | **Tipo:** PRIVADA SUSPEND  

Evalúa si una tarifa puede ser cobrada, compara versiones y calcula el nuevo balance.  
**Llama a `insertBDTransaction()` para el registro inicial.**

- Estado de Sincronización  
- Comparar Versiones  
- Validar Saldo  
- Calcular Balance  
- Registrar (llama a **insertBDTransaction()**)

---

### 3. insertBDTransaction (`private suspend fun`)
**Archivo:** `HomeController.kt` | **Tipo:** PRIVADA SUSPEND  

Inserta un registro de transacción en la base de datos local con todos los datos necesarios para sincronización posterior.  
**Es llamado internamente por `tariffEvaluator()`.**

---

### 4. registerPayment (`suspend fun`)
**Archivo:** `HomeController.kt` | **Tipo:** PÚBLICA SUSPEND  

Confirma el pago de la transacción y actualiza el balance en BD local.  
**Llama internamente a `saveNewBalance()` para actualizar el balance y notificar.**

---

### 5. saveNewBalance (`private suspend fun`)
**Archivo:** `HomeController.kt` | **Tipo:** PRIVADA SUSPEND  

Guarda el balance definitivo. Actualiza estado a **PAGADA**, recalcula, sincroniza versión y **Notifica vía RabbitMQ** (inicio de fase de sincronización).

---

## 📡 Sincronización (RabbitMQ)

Estas funciones gestionan la comunicación asíncrona entre validadores y el servidor, iniciando inmediatamente después de que `saveNewBalance()` publica un mensaje.

### 1. setupConsumer (`private fun`)
**Archivo:** `RabbitMqManager.kt` | **Tipo:** PRIVADA  

Configura el consumidor de mensajes de RabbitMQ para recibir actualizaciones de otros validadores y el servidor.

Ejemplo de mensaje JSON:
```json
{
  "u": "userId",
  "b": "balanceCurrent",
  "h": "hash",
  "s": "steep"
}

