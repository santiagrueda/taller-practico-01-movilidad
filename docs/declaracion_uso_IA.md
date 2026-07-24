# Declaración de uso de Inteligencia Artificial

Según el Pacto Pedagógico del curso, este taller permite y espera el uso de IA generativa para la
escritura de código. Se declara explícitamente qué partes se apoyaron en IA y qué fue validado por
el equipo.

## Qué se usó de la IA generativa (Claude / Claude Code)

- **Sintaxis de pandas:** carga de archivos, `pd.json_normalize` para aplanar el JSON de clima,
  `groupby`/`transform` para imputación por sensor+hora, `merge_asof` para integrar el log de clima,
  `pd.crosstab` para la tabla de contingencia, y el código de las visualizaciones con matplotlib.
- **Diagnóstico inicial de calidad de datos (GIGO):** exploración programática del archivo
  contaminado (nulos, duplicados, categorías, formatos de fecha, rangos de valores, coordenadas)
  para identificar y cuantificar los 7 problemas documentados en la Tarea 2.
- **Redacción de un primer borrador** de las justificaciones de limpieza, del análisis descriptivo,
  de la decisión de negocio final y de las respuestas del documento `Taller_Practico_01`.

## Qué fue decidido y validado por el equipo (no por la IA)

- La **estrategia de imputación** de `conteo_vehiculos` (mediana por sensor+hora, en vez de media o
  eliminación de filas) y el criterio de rango válido (`[0, 500]`).
- El **tratamiento de coordenadas inválidas** (imputar con la mediana histórica del propio sensor en
  vez de hacer swap automático ciego o descartar), incluyendo la trazabilidad explícita vía la
  columna `geo_corregida`.
- La **llave de negocio para duplicados** (`sensor_id` + `timestamp`), justificada con la evidencia
  de que cada sensor reporta una lectura por franja de 2 horas.
- La **interpretación de los resultados de la Parte 3.C** (que ni el tipo de vía ni el sensor
  individual explican la variación de tráfico, y que la señal real está en la hora del día) y la
  **recomendación de negocio final**, incluyendo el costo de falsos positivos/negativos.
- Toda cifra y conclusión fue verificada ejecutando el notebook de principio a fin y contrastando,
  solo como control de calidad (no como fuente de la limpieza), contra el archivo
  `movilidad_sensores_LIMPIO.csv` de referencia.

## Herramienta y versión

Claude (Anthropic), vía Claude Code, sesión del 24 de julio de 2026.
