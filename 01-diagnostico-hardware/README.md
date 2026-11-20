# Proyecto 01: Diagnóstico de almacenamiento en equipos reales

**Contexto:** diagnóstico del estado de salud, temperatura y rendimiento de los discos de equipos reales, con informes técnicos que justifican cada recomendación con datos medidos. Es el procedimiento que se aplica cuando un usuario reporta lentitud, apagados o riesgo de perder información.

**Herramientas:** CrystalDiskInfo 9.9.5 · CrystalDiskMark 9.0.3 · Información del sistema (`msinfo32`) · Calculadora de Windows (modo Programador)

**Habilidades que demuestra:**
- Lectura e interpretación de atributos SMART en discos NVMe
- Conversión de valores hexadecimales a decimal
- Pruebas térmicas en condiciones controladas (reposo, uso ligero, carga máxima)
- Medición de rendimiento y comparación con lo esperado según la interfaz (PCIe 3.0 y 4.0)
- Redacción de informes técnicos con diagnóstico y recomendaciones basadas en evidencia
- Protección de datos sensibles en evidencias (número de serie, nombre de equipo, usuario)

## Metodología

Cada disco se evalúa respondiendo cuatro preguntas, en este orden:

```
1. ¿Tiene fallas?          → filas 01, 0E, 0F       (si hay errores: respaldar antes de cualquier prueba)
2. ¿Cuánta vida le queda?  → filas 05, 03, 04
3. ¿Sufre por calor?       → filas 02, 12, 13, 10, 11, 1A a 1D
4. ¿Rinde lo que debe?     → CrystalDiskMark frente al rango esperado para su interfaz
                ↓
        Diagnóstico → Recomendaciones (cada una respaldada por un dato)
```

## Equipos evaluados

| Equipo | Disco | Interfaz | Salud | Hallazgo principal | Informe |
|---|---|---|---|---|---|
| 1. Laptop personal | Samsung MZVL8512HELU, 512 GB | PCIe 4.0 x4 | 93 % | Controlador a 89 °C con carga, sin reducción de velocidad; 56 apagados no seguros | [PDF](informe-equipo1.pdf) |
| 2. Dell OptiPlex 7400 AIO (laboratorio SENATI) | SK hynix BC711, 1 TB | PCIe 3.0 x4 | 100 % | Partición C: al 94 %; 346 apagados no seguros (30 % de los encendidos) | [PDF](informe-equipo2.pdf) |
| 3. Pendiente | (HDD) | | | | |

## Actividades realizadas

1. Inventario del equipo con `msinfo32`.
2. Captura de CrystalDiskInfo en reposo y revisión de errores **antes** de cualquier prueba de carga.
3. Conversión de los valores en bruto (hexadecimal) a decimal y de Kelvin a °C.
4. Prueba de carga con CrystalDiskMark (5 repeticiones de 4 GiB) y captura de CrystalDiskInfo inmediatamente después.
5. Comparación de temperaturas y contadores de reducción de velocidad (throttling) entre condiciones.
6. Redacción del informe: diagnóstico y recomendaciones vinculadas a datos concretos.

## Pruebas y resultado

| Indicador | Equipo 1 (PCIe 4.0) | Equipo 2 (PCIe 3.0) |
|---|---|---|
| Lectura secuencial (SEQ1M Q8T1) | 5029 MB/s | 3472 MB/s |
| Lectura de uso diario (RND4K Q1T1) | 60 MB/s | 44 MB/s |
| Temperatura compuesta en reposo / con carga | 46 °C / 63 °C | 36 °C / 45 °C |
| Reducción de velocidad por temperatura | Ninguna | 58 episodios leves (116 s en total), ninguno durante la prueba |

Los dos discos rinden dentro de lo esperado para su interfaz. Las evidencias están en los anexos de cada informe.

## Problemas que encontré y cómo los resolví

- **Prueba de carga inválida:** un video en 4K no exige al SSD, porque lo decodifica la tarjeta gráfica. Lo reemplacé por CrystalDiskMark.
- **Lecturas de un solo instante:** una diferencia de 2 °C entre dos capturas no es concluyente. Me basé en la temperatura compuesta y en los contadores del propio disco (filas 10, 11 y 1A a 1D).
- **Sensores distintos según el fabricante:** en el Samsung el sensor más caliente es el 1 y en el SK hynix el 2. No afirmo qué componente mide cada sensor si el fabricante no lo publica.
- **Alerta en rojo de CrystalDiskInfo:** es un límite del programa, no una alarma del disco. Lo verifiqué con los contadores del disco antes de concluir.
- **Datos sensibles en las capturas:** tapé el número de serie, el nombre del equipo y el usuario con rectángulos sólidos.

## Qué mejoraría en un entorno real

- Usar HWiNFO para registrar temperaturas mínimas, promedio y máximas durante toda la prueba, no una captura de un instante.
- En el Equipo 2, revisar en el Visor de eventos los eventos 41 y 6008 para identificar la causa de los apagados no seguros.
- Evaluar un equipo con disco mecánico (HDD) para cubrir los atributos 05, C5, C6 y C7.
- Comparar los resultados con las especificaciones oficiales del fabricante (velocidades y TBW).
