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

