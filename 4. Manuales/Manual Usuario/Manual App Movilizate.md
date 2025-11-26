# 📱 Wallet SDK API Documentation

## Documentación de la API del Wallet SDK de Flutter

### Wallet SDK API

Este SDK ha sido desarrollado en Flutter y está diseñado para integrarse fácilmente en aplicaciones móviles nativas (Android o iOS).  
Permite a las apps consumir servicios financieros relacionados con tarjetas, transacciones y operaciones de recarga o transferencia de saldo.

---

### 🧩 Objetivo

Facilitar la integración de funcionalidades de billetera digital en apps nativas sin necesidad de desarrollar componentes de UI o lógica desde cero.

---

### 🚀 Funcionalidades principales

- 🔍 Listar tarjetas vinculadas al usuario  
- 💳 Registrar nuevas tarjetas de crédito (incluye edición y eliminación)  
- 💸 Visualizar transacciones realizadas con las tarjetas  
- 💰 Consultar el saldo disponible por tarjeta  
- ➕ Recargar saldo de tarjeta con distintos métodos  
- 🏦 Recargar desde una entidad bancaria autorizada  
- 👤 Buscar usuarios receptores para realizar transferencias  
- 🔄 Modificar información de tarjetas ya registradas

### 📦 Tecnologías utilizadas

- Flutter como núcleo multiplataforma  
- Comunicación con la app nativa vía `MethodChannel`  
- Integración de servicios como Firebase, Push Notifications y conectividad
### 🔐 Autenticación

Todas las solicitudes requieren autenticación mediante token JWT en el header:

```http
Authorization: Bearer {token}
```
---
### 🔐 Auth | Actualizar parámetros del usuario

**Método:** PATCH  
**URL:** `https://dev-recaudo.clipp.app/auth/auth/update/{userId}/{deviceId}/{applicationId}/{version}`

---

#### Headers

| Campo         | Tipo   | Descripción                        |
|---------------|--------|------------------------------------|
| Content-Type  | String | `application/json; charset=UTF-8` |
| Authorization | String | Token JWT de autenticación         |
| locale        | String | Idioma de la aplicación            |

---

#### Parámetros

| Campo         | Tipo    | Descripción                      |
|---------------|---------|----------------------------------|
| userId        | String  | ID del usuario                   |
| deviceId      | String  | ID único del dispositivo         |
| applicationId | String  | ID de la aplicación              |
| version       | Number  | Versión de la aplicación         |
| body          | Object  | Objeto con los parámetros a actualizar |

---

#### Respuesta 200 (Success)

| Campo | Tipo   | Descripción            |
|-------|--------|------------------------|
| body  | Object | Cuerpo de la respuesta |

---

#### Errores 4xx

| Nombre       | Descripción               |
|--------------|---------------------------|
| Unauthorized | Usuario no autenticado    |
| UpdateFailed | Error al actualizar       |

---

#### Ejemplo de petición

```http 
PATCH https://dev-recaudo.clipp.app/auth/auth/update/{userId}/{deviceId}/{applicationId}/{version}
```
```http 
Headers:
Content-Type: application/json; charset=UTF-8
Authorization: Bearer {token}
locale: es

Body:
{
  "param1": "valor",
  "param2": "valor"
}
```
### 🔐 Auth | Actualizar preferencias compartidas

**Método:** PATCH  
**URL:** `https://dev-recaudo.clipp.app/auth/client/user/update-share/{userId}/{deviceId}`

---

#### Parámetros

| Campo        | Tipo    | Descripción                        |
|--------------|---------|------------------------------------|
| userId       | String  | ID del usuario                     |
| deviceId     | String  | ID único del dispositivo           |
| typeShared   | Number  | Tipo de preferencia compartida     |
| optionalData | Object  | Datos opcionales a guardar         |

---

#### Respuesta 200 (Success)

| Campo   | Tipo    | Descripción                              |
|---------|---------|------------------------------------------|
| success | Boolean | Preferencias actualizadas correctamente  |

---

#### Errores 4xx

| Nombre       | Descripción                          |
|--------------|--------------------------------------|
| UpdateFailed | Error al actualizar las preferencias |

---

#### Ejemplo de petición

```http
PATCH https://dev-recaudo.clipp.app/auth/client/user/update-share/{userId}/{deviceId}
```
```http
Body:
{
  "userId": "12345",
  "deviceId": "abcde",
  "typeShared": 1,
  "optionalData": {
    "theme": "dark",
    "notifications": true
  }
}
```
### 🔐 Auth | Actualizar token de notificaciones push

**Método:** PATCH  
**URL:** `https://dev-recaudo.clipp.app/auth/auth/update-token-push`

---

#### Headers

| Campo         | Tipo   | Descripción               |
|---------------|--------|---------------------------|
| Authorization | String | Token JWT de autenticación |

---

#### Parámetros

| Campo     | Tipo   | Descripción                                |
|-----------|--------|--------------------------------------------|
| tokenPush | String | Token de notificaciones push del dispositivo |

---

#### Respuesta 200 (Success)

| Campo   | Tipo    | Descripción                     |
|---------|---------|---------------------------------|
| success | Boolean | Token actualizado correctamente |
| message | String  | Mensaje de confirmación         |

---

#### Errores 4xx

| Nombre       | Descripción                        |
|--------------|------------------------------------|
| InvalidToken | Token de push inválido             |
| UpdateFailed | Error al actualizar el token       |

---

#### Ejemplo de petición

```http
PATCH https://dev-recaudo.clipp.app/auth/auth/update-token-push
```
```http
Headers:
Authorization: Bearer {token}

Body:
{
  "tokenPush": "abc123xyz456"
}
```
### 🔐 Auth | Crear un enlace dinámico

**Método:** POST  
**URL:** `https://dev-recaudo.clipp.app/ad/client/link/create/{userId}/{deviceId}/{applicationId}/{countryId}/{cityId}/{version}`

---

#### Parámetros

| Campo         | Tipo    | Descripción               |
|---------------|---------|---------------------------|
| userId        | String  | ID del usuario            |
| deviceId      | String  | ID único del dispositivo  |
| applicationId | String  | ID de la aplicación       |
| countryId     | Number  | ID del país               |
| cityId        | Number  | ID de la ciudad           |
| version       | Number  | Versión de la aplicación  |
| body          | Object  | Parámetros para el enlace |

**Body (objeto):**

| Campo         | Tipo    | Descripción               |
|---------------|---------|---------------------------|
| appId         | String  | ID de la app              |
| applicationId | String  | ID de la aplicación       |
| countryId     | Number  | ID del país               |
| cityId        | Number  | ID de la ciudad           |
| typeEvent     | Number  | Tipo de evento            |
| title         | String  | Título del enlace         |
| description   | String  | Descripción del enlace    |
| key           | Object  | Datos clave               |
| image         | String  | URL de la imagen (opcional) |

---

#### Respuesta 200 (Success)

| Campo | Tipo   | Descripción              |
|-------|--------|--------------------------|
| link  | String | Enlace dinámico creado   |

---

#### Errores 4xx

| Nombre            | Descripción               |
|-------------------|---------------------------|
| LinkCreationFailed| Error al crear el enlace  |

---

#### Ejemplo de petición

```http
POST https://dev-recaudo.clipp.app/ad/client/link/create/{userId}/{deviceId}/{applicationId}/{countryId}/{cityId}/{version}
```
```http
Body:
{
  "appId": "mobilize-app",
  "applicationId": "123",
  "countryId": 593,
  "cityId": 11,
  "typeEvent": 1,
  "title": "Promoción especial",
  "description": "Viaja con descuento",
  "key": {
    "promoCode": "DESC10"
  },
  "image": "https://example.com/banner.png"
}
```
### 🔐 Auth | Enviar código SMS de verificación v2

**Método:** POST  
**URL:** `https://dev-recaudo.clipp.app/auth/client/user/send-sms-v2/{countryCode}/{phone}/{userId}/{deviceId}/{applicationId}/{link}/{version}`

---

#### Parámetros

| Campo         | Tipo    | Descripción               |
|---------------|---------|---------------------------|
| countryCode   | String  | Código de país del teléfono |
| phone         | String  | Número de teléfono a verificar |
| userId        | String  | ID del usuario            |
| deviceId      | String  | ID único del dispositivo  |
| applicationId | String  | ID de la aplicación       |
| link          | String  | Enlace para autocompletar |
| version       | Number  | Versión de la aplicación  |

---

#### Respuesta 200 (Success)

| Campo   | Tipo    | Descripción               |
|---------|---------|---------------------------|
| success | Boolean | SMS enviado correctamente |

---

#### Errores 4xx

| Nombre       | Descripción                  |
|--------------|------------------------------|
| InvalidPhone | Número de teléfono inválido  |

---

#### Ejemplo de petición

```http
POST https://dev-recaudo.clipp.app/auth/client/user/send-sms-v2/{countryCode}/{phone}/{userId}/{deviceId}/{applicationId}/{link}/{version}
```
### 🔐 Auth | Enviar código SMS de verificación

**Método:** POST  
**URL:** `https://dev-recaudo.clipp.app/auth/client/user/send-sms/{countryCode}/{phone}/{userId}/{deviceId}/{applicationId}/{version}`

---

#### Headers

| Campo         | Tipo   | Descripción               |
|---------------|--------|---------------------------|
| Authorization | String | Token JWT de autenticación |

---

#### Parámetros

| Campo         | Tipo    | Descripción               |
|---------------|---------|---------------------------|
| countryCode   | String  | Código de país del teléfono |
| phone         | String  | Número de teléfono a verificar |
| userId        | String  | ID del usuario            |
| deviceId      | String  | ID único del dispositivo  |
| applicationId | String  | ID de la aplicación       |
| version       | Number  | Versión de la aplicación  |

---

#### Respuesta 200 (Success)

| Campo      | Tipo    | Descripción                              |
|------------|---------|------------------------------------------|
| success    | Boolean | SMS enviado correctamente                |
| message    | String  | Mensaje de confirmación                  |
| expiresIn  | Number  | Tiempo de expiración del código en segundos |

---

#### Errores 4xx

| Nombre            | Descripción                  |
|-------------------|------------------------------|
| InvalidPhone      | Número de teléfono inválido  |
| SmsLimitExceeded  | Límite de SMS excedido       |
| SmsServiceError   | Error en el servicio de SMS  |

---

#### Ejemplo de petición

```http
POST https://dev-recaudo.clipp.app/auth/client/user/send-sms/{countryCode}/{phone}/{userId}/{deviceId}/{applicationId}/{version}
```
Headers:
Authorization: Bearer {token}
Body:
{
  "countryCode": "+593",
  "phone": "987654321",
  "userId": "12345",
  "deviceId": "abcde",
  "applicationId": "mobilize-app",
  "version": 2
}
### 🔐 Auth | Login con Mobilize

**Método:** POST  
**URL:** `https://dev-recaudo.clipp.app/auth/auth/application/{deviceId}/{applicationId}/{version}`

---

#### Parámetros

| Campo         | Tipo    | Descripción               |
|---------------|---------|---------------------------|
| deviceId      | String  | ID único del dispositivo  |
| applicationId | String  | ID de la aplicación       |
| version       | Number  | Versión de la aplicación  |
| body          | Object  | Datos del usuario         |

**Body (objeto):**

| Campo         | Tipo    | Descripción               |
|---------------|---------|---------------------------|
| email         | String  | Correo electrónico del usuario |
| firstName     | String  | Nombre del usuario        |
| lastName      | String  | Apellido del usuario      |
| idDevice      | String  | ID del dispositivo        |
| tokenPush     | String  | Token de notificaciones push |
| idApplication | String  | ID de Mobilize            |
| application   | Object  | Objeto de la aplicación   |
| application.id| String  | ID de la aplicación       |
| idApp         | String  | ID de la app              |
| countryCode   | String  | Código de país            |
| phone         | String  | Número de teléfono        |
| phoneVerified | Boolean | Teléfono verificado       |

---

#### Respuesta 200 (Success)

| Campo             | Tipo    | Descripción               |
|-------------------|---------|---------------------------|
| user              | Object  | Datos del usuario         |
| user.id           | String  | ID único del usuario      |
| user.firstName    | String  | Nombre del usuario        |
| user.lastName     | String  | Apellido del usuario      |
| user.email        | String  | Correo electrónico        |
| user.phone        | String  | Número de teléfono        |
| user.countryCode  | String  | Código de país del usuario |

---

#### Errores 4xx

| Nombre      | Descripción          |
|-------------|----------------------|
| LoginFailed | Error en el login    |

---

#### Ejemplo de petición

```http
POST https://dev-recaudo.clipp.app/auth/auth/application/{deviceId}/{applicationId}/{version}
```
Body:
{
  "email": "usuario@example.com",
  "firstName": "Juan",
  "lastName": "Pérez",
  "idDevice": "abcde12345",
  "tokenPush": "pushTokenXYZ",
  "idApplication": "mobilize-001",
  "application": {
    "id": "app-123"
  },
  "idApp": "mobilize-app",
  "countryCode": "+593",
  "phone": "987654321",
  "phoneVerified": true
}
