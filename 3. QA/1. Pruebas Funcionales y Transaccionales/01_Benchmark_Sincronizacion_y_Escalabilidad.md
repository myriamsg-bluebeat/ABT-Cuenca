# Informe rendimiento en 1 equipo

**Dispositivo:** Validador Offline (IoT)  
**Estado:** Pruebas de carga e inserción

Este documento detalla los tiempos de descarga, procesamiento e inserción en base de datos local, así como el análisis de consumo de ancho de banda e infraestructura para una unidad individual bajo el escenario de prueba actual.

---

## 1. Resumen de Métricas por Entidad

| Entidad (Dataset) | Registros | Peso Total | Peso Promedio (Unitario) |
| :--- | :--- | :--- | :--- |
| **DNI / Tarjetas** | 86,087 | 12.25 MB | 0.000142 MB (142 bytes) |
| **Balances (Saldos)** | 89,828 | 6.58 MB | 0.000073 MB (73 bytes) |
| **Tarifas** | 5 | 0.067 MB | 0.0134 MB (13.4 KB) |

---

## 2. Análisis de Throughput (Velocidad de Procesamiento)

Medición del tiempo total que toma el equipo en descargar e insertar la data en su almacenamiento local.

### **Balances (Saldos)** `Rápido`
* **Tiempo Total:** 132.2 seg
* **Duración:** ~2 min 12s
* **Velocidad:** **679 registros/seg**

### **DNI / Tarjetas** `Lento`
* **Tiempo Total:** 233.4 seg
* **Duración:** ~3 min 53s
* **Velocidad:** **369 registros/seg**

### **Tarifas**
* **Tiempo Total:** 1.07 seg
* **Duración:** Instantánea
* **Velocidad:** **Instantánea**

---

## 3. Requisitos de Infraestructura y Red (Escenario de Prueba Actual)

Datos consolidados del consumo real para un solo equipo procesando la carga actual (~176k registros combinados).

| Métrica | Valor Calculado (1 Equipo) | Descripción |
| :--- | :--- | :--- |
| **Volumen Total de Datos** | **18.90 MB** | Espacio de almacenamiento ocupado tras la sincronización (Suma de DNI, Balances y Tarifas). |
| **Tiempo Total de Operación** | **366.67 Segundos** | Tiempo total que el equipo tarda en descargar e insertar toda la información (~6.1 minutos). |
| **Ancho de Banda Requerido** | **0.052 MB/s** | Velocidad mínima de red necesaria para que la descarga no frene al procesador del equipo. |

> **Nota Técnica:** Para este volumen de datos (no masivo), el requerimiento de red por equipo individual es bajo (**~52 KB/s**), ya que el limitante principal es la velocidad de escritura en la base de datos del dispositivo, no la conexión a internet.


# Análisis para 1 millón de datos en 1 equipo

**Escenario:** Proyección de Carga Masiva (1 Millón) | **Alcance:** Unidad Individual

Este documento proyecta los tiempos de descarga, procesamiento y almacenamiento requeridos para un solo equipo validador cuando la base de datos escala a 1 millón de registros, aislando el rendimiento del hardware sin considerar la concurrencia de la flota.

---

## 1. Proyección de Métricas por Entidad

| Entidad (Dataset) | Registros | Peso Total Estimado | Peso Promedio (Unitario) |
| :--- | :--- | :--- | :--- |
| **DNI / Tarjetas** | 1,000,000 | 142.25 MB | 0.000142 MB (142 bytes) |
| **Balances (Saldos)** | 1,000,000 | 73.28 MB | 0.000073 MB (73 bytes) |
| **Tarifas** | 5 | 0.07 MB | 0.0134 MB (13.4 KB) |

---

## 2. Estimación de Tiempos de Procesamiento

Cálculo del tiempo que el equipo permanecerá ocupado ("Ventana de Mantenimiento") basándose en la velocidad de escritura actual.

### **Balances (Saldos)**
* **Cantidad:** 1 Millón
* **Tiempo Estimado:** ~24.5 minutos
* **Velocidad Base:** 679 registros/seg

### **DNI / Tarjetas**
* **Cantidad:** 1 Millón
* **Tiempo Estimado:** ~45.2 minutos
* **Velocidad Base:** 369 registros/seg

### **Tarifas**
* **Tiempo Estimado:** < 1 segundo (Instantáneo)

---

## 3. Requisitos de Infraestructura y Red (Escenario 1 Millón)

Recursos totales que consumirá **un solo equipo** para completar la carga masiva.

| Métrica | Valor Proyectado (1 Equipo) | Descripción |
| :--- | :--- | :--- |
| **Volumen Total de Datos** | **215.60 MB** | Espacio de almacenamiento local requerido para alojar la base de datos completa. |
| **Tiempo Total de Operación** | **69.72 Minutos** | Tiempo total (~1h 10m) que el equipo tarda en procesar el millón de registros secuencialmente. |
| **Ancho de Banda Requerido** | **0.052 MB/s** | Velocidad mínima de descarga para mantener el flujo de datos constante hacia el procesador. |

> **Nota de Escalabilidad:** Aunque el volumen de datos aumenta drásticamente (de 18 MB a 215 MB), el ancho de banda requerido por el equipo individual (**~52 KB/s**) se mantiene bajo y constante, ya que está limitado por la velocidad física a la que el procesador puede guardar los datos, no por la velocidad de la red.



---

# Analisis con flota 420 equipos con 1 millon de datos

**Escenario:** Despliegue Masivo Simultáneo | **Alcance:** Flota Completa (420 Unidades)

Este apartado escala los requisitos individuales a nivel de servidor, calculando el impacto en la red si la flota completa de 420 buses intenta descargar la base de datos de 1 millón de registros al mismo tiempo (ej. evento de "Big Bang" o actualización crítica).

---

## 1. Volumen Total de Información (Carga de Red)

Cálculo de la cantidad de datos que el servidor debe "escupir" (Data Egress) a la red para abastecer a todos los equipos.

* **Fórmula:** Volumen Unitario (215.60 MB) $\times$ Cantidad de Equipos (420)
* **Total en Megabytes:** $90,552 \text{ MB}$
* **Conversión a Gigabytes:** $90,552 / 1,024 \approx \mathbf{88.43 \text{ GB}}$

> **Impacto:** El servidor transferirá casi 90 GB de datos en un solo evento de sincronización masiva.

---

## 2. Ancho de Banda del Servidor Requerido

Para evitar cuellos de botella, la red del servidor debe ser capaz de entregar los datos a la misma velocidad que los equipos los procesan (4,183 segundos).

* **Volumen a Transferir:** $90,552 \text{ MB}$
* **Ventana de Tiempo (Processing Time):** $4,183.2 \text{ s}$ (~69.7 min)
* **Cálculo de Throughput:** $90,552 / 4,183.2 = \mathbf{21.65 \text{ MB/s}}$

### Resultado de Velocidad
El servidor requiere un ancho de banda de salida dedicado de **21.65 MB/s** (aprox. 173 Mbps) constantes durante 1 hora y 10 minutos para que la red no sea el factor limitante.

---

## 3. Resumen Ejecutivo de Infraestructura (Servidor)

Tabla consolidada de recursos necesarios en el lado del servidor para soportar este escenario.

| Concepto | Valor Calculado | Unidad |
| :--- | :--- | :--- |
| **Tráfico de Salida Total** | **88.43** | **GB** |
| **Velocidad de Subida Mínima** | **21.65** | **MB/s** |
| **Conexiones Concurrentes** | **420** | **Sockets** |
| **Tiempo de Saturación de Red** | **~70** | **Minutos** |

# Diferencia de cargas entre 1 equipo y 420 equipos

**Comparativa:** Impacto Unitario vs. Estrés de Servidor

Esta sección visualiza el factor de multiplicación que sufre la infraestructura. Mientras que para un solo equipo la operación es trivial, la concurrencia de 420 unidades transforma una descarga ligera en un evento de alto estrés para la red (Thundering Herd Problem).

---

## 1. Tabla Comparativa de Recursos

Comparación directa de los recursos necesarios para completar la sincronización de 1 millón de registros en el mismo periodo de tiempo (69.72 minutos).

| Métrica Crítica | Carga Unitaria (1 Equipo) | Carga de Flota (420 Equipos) | Factor de Multiplicación |
| :--- | :--- | :--- | :--- |
| **Volumen Total (Storage)** | 215.60 MB | **88.43 GB** | $\times 420$ |
| **Ancho de Banda (Network)** | 0.052 MB/s | **21.65 MB/s** | $\times 420$ |
| **Conexiones Activas** | 1 Socket | **420 Sockets** | $\times 420$ |
| **Tiempo de Proceso** | ~70 Minutos | ~70 Minutos | Igual (Limitado por HW local) |

---

## 2. Interpretación del Impacto en Infraestructura

### A. Perspectiva del Validador (El Cliente)
Para un equipo individual, la tarea es **intensiva en CPU/Disco** pero ligera en red.
* El equipo descarga a una velocidad muy baja (**52 KB/s**) porque su "cuello de botella" es la velocidad a la que puede escribir en su memoria interna.
* *Impacto:* El validador ni siquiera satura una conexión 3G básica.

### B. Perspectiva del Servidor (El Backend)
Para el servidor, la tarea es **intensiva en Red (I/O)**.
* El servidor debe sostener **420 flujos simultáneos**. Aunque cada flujo es lento individualmente, la suma obliga al servidor a tener una salida robusta de **21.65 MB/s**.
* *Riesgo:* Si el servidor no tiene ese ancho de banda, los paquetes se encolan, causando *timeouts* en los validadores y reinicios de descarga, lo que podría extender la ventana de mantenimiento de 70 minutos a varias horas.

> **Conclusión de Escalabilidad:** El sistema escala linealmente en consumo de red. Cada nuevo bus agregado al sistema añade directamente 215 MB de carga al servidor en eventos de sincronización total.


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
