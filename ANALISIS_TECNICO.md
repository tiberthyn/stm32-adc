# 🔬 Análisis Técnico: ADC + TIM2 + DMA en STM32

> Fundamentos matemáticos y de timing para entender el subsistema de adquisición.  
> **Objetivo**: Validar que la configuración actual es viable y explorar márgenes de optimización.

---

## 📋 Especificaciones del Sistema

| Parámetro | Valor | Fuente |
|-----------|-------|--------|
| `SYSCLK` | 80 MHz | `SystemClock_Config()` |
| `ADC Clock` | 80 MHz (ASYNC_DIV1) | `hadc1.Init.ClockPrescaler` |
| `FS_HZ` (frecuencia de muestreo) | 500,000 Hz | `#define FS_HZ` |
| `N_CHANNELS` | 2 | `#define N_CHANNELS` |
| `Resolution` | 12 bits | `ADC_RESOLUTION_12B` |
| `SamplingTime` | 47.5 ciclos ADC | `ADC_SAMPLETIME_47CYCLES_5` |
| `TIM2 Period` | 1 (conteo 0→1) | `htim2.Init.Period = 1` |
| `TIM2 Prescaler` | 79 | `htim2.Init.Prescaler = 79` |

---

## ⚙️ 1. Timing del Timer TIM2 (Trigger Source)

### 🔢 Cálculo de la frecuencia de trigger
f_TIM2_clk = SYSCLK / (Prescaler + 1)
= 80 MHz / (79 + 1)
= 80 MHz / 80
= 1 MHz → T_tick = 1 µs

f_trigger = f_TIM2_clk / (Period + 1)
= 1 MHz / (1 + 1)
= 500 kHz → T_trigger = 2 µs


✅ **Conclusión**: TIM2 genera un evento de trigger cada **2 µs**, exactamente la frecuencia de muestreo deseada (`FS_HZ = 500 kHz`).

### 🎯 ¿Por qué usar TRGO y no software trigger?

| Método | Jitter típico | Carga CPU | Determinismo |
|--------|--------------|-----------|--------------|
| Software (`HAL_ADC_Start()`) | 5-50 µs | Alta | ❌ Variable |
| Hardware (`TIM2 TRGO`) | < 10 ns | Nula | ✅ Determinista |

> 📌 **Fundamento**: El trigger hardware bypassa la latencia de interrupciones y scheduling del núcleo, esencial para adquisición sincronizada.

---

## ⚡ 2. Timing del ADC1 (Conversión Real)

### 🔁 Ciclo completo por muestra (2 canales en scan mode)

El ADC en modo scan con 2 canales ejecuta secuencialmente:

[Trigger] → [Sample CH8] → [Convert CH8] → [Sample CH4] → [Convert CH4] → [DMA Write] → [EOC/Interrupt]


### 🔢 Cálculo del tiempo mínimo por secuencia (2 canales)
T_sample = SamplingTime + 12.5 ciclos (conversión 12-bit SAR)
= 47.5 + 12.5 = 60 ciclos ADC

T_secuencia = N_channels × T_sample
= 2 × 60 ciclos = 120 ciclos ADC

T_secuencia_real = 120 ciclos / f_ADC_clk
= 120 / 80 MHz
= 1.5 µs


### ✅ Verificación de viabilidad
Requerido: T_trigger = 2.0 µs (500 kHz)
Disponible: T_secuencia = 1.5 µs
Margen = T_trigger - T_secuencia = 2.0 - 1.5 = 0.5 µs (25% de holgura)


✅ **Conclusión**: El ADC **puede completar** la conversión de 2 canales antes del siguiente trigger. Hay un **margen del 25%**.

### ⚠️ Advertencia: Overrun

Si `T_secuencia > T_trigger`, el ADC genera error `ADC_OVR_DATA_OVERWRITTEN`.  
En nuestro caso: `1.5 µs < 2.0 µs` → ✅ Sin riesgo de overrun.

---

## 🚚 3. DMA: Transferencia y Throughput

### 🔢 Cálculo de ancho de banda requerido
Datos por muestra: 2 canales × 16 bits (uint16_t) = 4 bytes
Frecuencia: 500,000 muestras/seg
Throughput requerido = 500,000 × 4 bytes/seg = 2,000,000 B/s = 2 MB/s


### 🔌 Capacidad del DMA en STM32L4

| Parámetro | Valor |
|-----------|-------|
| DMA Clock | HCLK = 80 MHz |
| Transferencia por ciclo | 1 palabra (16/32 bits) |
| Throughput teórico máximo | ~40-80 MB/s (dependiendo de burst) |

✅ **Conclusión**: 2 MB/s requeridos ≪ 40 MB/s disponibles → **DMA no es cuello de botella**.

### 🔄 ¿Circular o Normal?

En el código actual se usa **DMA Normal** (no circular):

```c
// En MX_ADC1_Init():
hadc1.Init.DMAContinuousRequests = ENABLE;  // DMA pide datos continuamente
// Pero StartCapture() usa HAL_ADC_Start_DMA() con tamaño fijo → modo Normal

