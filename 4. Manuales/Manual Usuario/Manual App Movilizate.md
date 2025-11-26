# 📱 Wallet SDK API Documentation

Documentación de la API del Wallet SDK desarrollado en Flutter para integración en apps nativas Android/iOS.

---

## 🧩 Objetivo

Facilitar la integración de funcionalidades de billetera digital en apps nativas sin necesidad de desarrollar componentes de UI o lógica desde cero.

---

## 🚀 Funcionalidades principales

- 🔍 Listar tarjetas vinculadas al usuario  
- 💳 Registrar, editar y eliminar tarjetas de crédito  
- 💸 Visualizar transacciones realizadas  
- 💰 Consultar saldo disponible por tarjeta  
- ➕ Recargar saldo con distintos métodos  
- 🏦 Recargar desde entidades bancarias autorizadas  
- 👤 Buscar usuarios receptores para transferencias  
- 🔄 Modificar información de tarjetas registradas  

---

## 📦 Tecnologías utilizadas

- Flutter como núcleo multiplataforma  
- Comunicación con la app nativa vía `MethodChannel`  
- Integración con Firebase, Push Notifications y conectividad  

---

## 🔧 Autenticación

Todas las solicitudes requieren autenticación mediante token JWT en el header:

```http
Authorization: Bearer {token}

