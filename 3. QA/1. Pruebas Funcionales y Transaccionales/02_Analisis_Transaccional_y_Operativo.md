
# Análisis de Rendimiento Transaccional (Unitario)

**Alcance:** Ciclo de Vida de 1 Transacción | **Tipo:** Tiempo Real

Este apartado analiza el comportamiento del equipo durante el proceso de cobro (tarjeta "Movilizate", QR o Mifare). Se mide el peso del paquete de subida y la latencia desde que se envía la petición al servidor hasta que se recibe la confirmación (ACK).

---

## 1. Muestreo de Transacciones Reales

Datos obtenidos de trazas de red en operación real, calculando la diferencia exacta entre el *timestamp* de envío y recepción.

| Muestra | Tamaño (Payload) | Tiempo Inicio | Tiempo Fin | Latencia (Duración) |
| :--- | :--- | :--- | :--- | :--- |
| **Transacción 1** | 3 KB | 17:08:22.396 | 17:08:22.649 | **0.253 seg** (253 ms) |
| **Transacción 2** | 2 KB | 17:08:56.173 | 17:08:56.384 | **0.211 seg** (211 ms) |
| **Transacción 3** | 2 KB | - | - | - |

---

## 2. Resumen de Métricas por Cobro

Promedios calculados para proyectar el consumo de recursos por cada pasajero que aborda el bus.

| Métrica | Valor Promedio | Observación Técnica |
| :--- | :--- | :--- |
| **Tamaño de Subida** | **~ 2.3 KB** | Incluye datos de tarjeta, fecha, monto y cabeceras de seguridad. |
| **Latencia de Red** | **~ 232 ms** | Tiempo que el bus espera confirmación del servidor. |
| **Velocidad Requerida** | **~ 9.9 KB/s** | Pico de ancho de banda momentáneo durante el cobro. |

---

## 3. Interpretación de Rendimiento

* **Eficiencia:** El tamaño del paquete (2-3 KB) está altamente optimizado. Es lo suficientemente pequeño para ser transmitido incluso en redes 2G/EDGE inestables.
* **Velocidad de Usuario:** Una latencia de red de **0.23 segundos** es imperceptible para el usuario final. El sistema responde prácticamente en tiempo real.
* **Consumo de Datos:** Una sola transacción consume apenas **0.0023 MB**.
    * *Ejemplo:* Se necesitarían **434 transacciones** para consumir apenas **1 MB** de datos del plan móvil del equipo.

    


# Análisis de Operación Transaccional Diaria (Flota Completa)

**Escenario:** Operación Continua (Run-time) | **Alcance:** Tráfico generado por uso real del sistema

Este apartado consolida el comportamiento de la red durante un día típico. Se desglosan paso a paso los volúmenes de datos generados por la interacción de los pasajeros, desde la transacción individual hasta la réplica masiva a toda la flota.

---

## 1. Definición de Variables Operativas

Base de cálculo utilizada para las proyecciones de tráfico.

| Variable | Valor Base | Origen del Dato |
| :--- | :--- | :--- |
| **Tamaño de Flota** | **420** Equipos | Total de validadores activos en el sistema. |
| **Uso por Equipo** | **600** Tx/día | Promedio estimado de pasajeros por bus al día. |
| **Transacciones Totales** | **252,000** Tx/día | Cálculo: $420 \text{ buses} \times 600 \text{ tx}$. |
| **Peso por Transacción** | **2.3 KB** | Promedio medido en pruebas unitarias (Subida). |
| **Peso por Cambio de Saldo** | **73 Bytes** | Tamaño del registro de balance (Deltas). |

---

## 2. Proyección de Volumen de Datos (24 Horas)

Cálculo detallado del tráfico que entra y sale del servidor en un día completo.

| Flujo de Datos | Fórmula Desglosada | Volumen Total Diario |
| :--- | :--- | :--- |
| **1. Generación (Subida)**<br>*(De Buses a Servidor)* | $252,000 \text{ tx} \times 2.3 \text{ KB}$ | **579.6 MB** |
| **2. Archivo de Deltas**<br>*(Consolidado en Servidor)* | $252,000 \text{ cambios} \times 73 \text{ Bytes}$ | **18.4 MB** (Base de cambios global) |
| **3. Réplica (Bajada)**<br>*(De Servidor a Buses)* | $18.4 \text{ MB} \times 420 \text{ buses}$ | **7,728 MB (~7.55 GB)** |
| **TOTAL TRÁFICO** | Subida (1) + Réplica (3) | **~ 8.1 GB / Día** |

> **Nota sobre la Réplica:** Aunque los buses solo suben ~0.5 GB de datos (las transacciones), el servidor debe enviar ~7.5 GB de vuelta. Esto ocurre porque **cada uno de los 420 buses** debe descargar el archivo de 18.4 MB que contiene los nuevos saldos de las 252,000 tarjetas usadas en el día.

---

## 3. Análisis de Estrés: Hora Pico (Peak Hour)

Para dimensionar el ancho de banda, asumimos el peor escenario: la "Hora Punta" (ej. 7:00 AM - 8:00 AM), donde se concentra el tráfico más denso.

* **Factor de Concentración:** Se estima que el **20%** de todas las transacciones diarias ocurren en esta hora.
* **Cálculo de Volumen Hora Pico:** $252,000 \text{ tx} \times 20\% = \mathbf{50,400 \text{ tx}}$.
* **Tasa por Segundo (TPS):** $50,400 \text{ tx} / 3,600 \text{ seg} = \mathbf{14 \text{ tx/s}}$.

### Requerimiento de Ancho de Banda (Tiempo Real)

Velocidad de red necesaria en el servidor para procesar estas 14 transacciones por segundo y notificar a la flota.

| Dirección | Descripción del Tráfico | Velocidad Requerida |
| :--- | :--- | :--- |
| **Entrada (Ingress)** | Recibir 14 tx/s $\times$ 2.3 KB | **0.032 MB/s** (Mínimo) |
| **Salida (Egress)** | Enviar confirmación a 420 buses (Broadcast)*<br>*(14 tx/s $\times$ 2 KB $\times$ 420 destinos)* | **11.76 MB/s** (Crítico) |

*\*Este cálculo de salida asume que el sistema intenta notificar el nuevo saldo a todos los buses casi en tiempo real. Si la actualización se hace por lotes cada 5 minutos, este requerimiento baja drásticamente.*

---

## 4. Conclusiones de Operación Diaria

1.  **Origen del Tráfico:** El número masivo de **252,000 transacciones** nace de multiplicar la operatividad individual (600 pasajeros) por la flota total.
2.  **Asimetría de Red:** El sistema consume **15 veces más bajada que subida** desde la perspectiva de los buses. El servidor actúa principalmente como un difusor de datos (Broadcaster).
3.  **Capacidad del Servidor:** Para soportar la hora pico sin que se acumulen colas de procesamiento, el servidor debe garantizar una salida constante de **~12 MB/s** (aprox. 100 Mbps dedicados) exclusivamente para transacciones.

