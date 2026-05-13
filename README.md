# 🔬 STM32 ADC Project

> Proyecto educativo para entender la adquisición de señales con STM32, usando ADC + DMA + Timer trigger.

## 🎯 Objetivos de Aprendizaje
- Comprender la sincronización hardware entre Timer y ADC
- Entender el flujo de datos vía DMA sin intervención de CPU
- Analizar el timing real de conversión ADC en STM32L4/L5
- Visualizar el impacto de parámetros de configuración en la adquisición

## 📁 Estructura del Repositorio
📦 stm32-adc  
├── 📄 README.md ← Este archivo  
├── 📄 ANALISIS_TECNICO.md ← Análisis matemático ADC/TIM2/DMA ⭐  
├── 📄 main.c ← Firmware principal  
├── 📁 scripts/ ← Herramientas Python para post-procesamiento  
└── 📁 docs/ ← Diagramas y recursos visuales

## 🧪 Especificaciones del Firmware
| Parámetro | Valor | Descripción |
|-----------|-------|-------------|
| `FS_HZ` | 500,000 | Frecuencia de muestreo objetivo |
| `CAPTURE_MS` | 5 | Duración de cada captura |
| `N_CHANNELS` | 2 | Canales ADC simultáneos (entrada/salida) |
| `Resolution` | 12-bit | Rango 0-4095 |
| `Trigger` | TIM2 TRGO | Sincronización hardware |
| `Transfer` | DMA Circular | Sin carga en CPU |

## 🚀 Cómo Usar
1. Abrir `main.c` en STM32CubeIDE o VS Code
2. Compilar para tu target (STM32L4/L5 recomendado)
3. Flash con ST-Link
4. Conectar terminal serial @ 460800 baudios para ver datos
5. (Opcional) Usar `scripts/parse_uart.py` para decodificar binario

## 📚 Recursos Educativos
- [🔗 ANALISIS_TECNICO.md](./ANALISIS_TECNICO.md) ← **Lectura obligatoria**
- [🔗 Datasheet STM32L4](https://www.st.com/resource/en/datasheet/stm32l4r5zi.pdf)
- [🔗 Reference Manual RM0351](https://www.st.com/resource/en/reference_manual/rm0351-stm32l4x5-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)

## 🤝 Contribuciones
¡Este proyecto es educativo! Si encuentras errores o tienes mejoras:
1. Fork del repositorio
2. Crea una rama (`git checkout -b feature/mejora`)
3. Commit tus cambios (`git commit -m 'Add: explicación de X'`)
4. Push a la rama (`git push origin feature/mejora`)
5. Abre un Pull Request

## 📄 Licencia
MIT License - ver [LICENSE](LICENSE) para detalles.
