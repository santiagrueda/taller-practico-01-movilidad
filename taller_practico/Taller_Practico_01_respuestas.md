# Taller Práctico 01 — Respuestas

**Conjunto de datos elegido:** C — Movilidad urbana (`movilidad_sensores` + `clima_api_log.json`)

> Nota metodológica: el equipo optó por responder en Markdown (opción 2 de la sección 6.1 de la
> Guía del Taller), en vez de compilar el `.tex`, ya que no se contaba con una distribución LaTeX
> instalada localmente y la Guía permite ambas alternativas de forma indistinta. El `.tex` original
> se conserva en este mismo directorio como referencia del enunciado recibido.

---

## Parte 1 — Análisis estadístico general (sobre el archivo LIMPIO)

### 1.1 — Taxonomía

| Variable | Clasificación | Por qué esa clasificación determina el estadístico válido |
|---|---|---|
| `sensor_id` | Nominal (id) | Es una etiqueta sin orden ni magnitud; no tiene sentido calcular un "promedio de sensor", solo se puede contar frecuencia o agrupar por él. |
| `tipo_via` | Nominal (decisión del equipo, discutible como ordinal) | Aunque existe una jerarquía vial intuitiva (Troncal > Arteria > Local en capacidad), el dataset no define una escala numérica asociada a esa jerarquía, así que tratarla como nominal evita asumir una distancia "igual" entre categorías que no está medida. Si se tratara como ordinal, la moda o mediana de posición seguirían siendo válidas, pero nunca la media. |
| `timestamp` | Fecha-hora | Es una escala temporal continua con un punto de referencia arbitrario (no absoluto); se pueden calcular diferencias (duraciones) pero no razones ("el doble de fecha" no tiene sentido), por lo que se resume con conteos por franja, no con media aritmética directa del valor. |
| `conteo_vehiculos` | Discreta | Solo toma valores enteros no negativos (no existen 22.5 vehículos); admite media y mediana, pero al ser un conteo de eventos con distribución sesgada, la mediana es más representativa del valor típico. |
| `temperatura_c` | Continua | Puede tomar cualquier valor real dentro de un rango físico; admite media, desviación estándar y todas las operaciones aritméticas propias de una escala de intervalo. |

### 1.2 — Medidas de tendencia central y dispersión

Variable ordinal (discutida como tal): **`tipo_via`**. Variable continua: **`temperatura_c`**.

a) **Tendencia central:**
   - `tipo_via` (ordinal/categórica): usaríamos la **moda** (categoría más frecuente), nunca la media,
     porque no existe una operación de suma/promedio válida entre "Troncal", "Arteria" y "Local" — el
     promedio numérico de códigos asignados arbitrariamente (ej. 1, 2, 3) no representaría nada real
     sobre el tipo de vía.
   - `temperatura_c`: usaríamos la **media** (23.1 °C), porque su distribución es razonablemente
     simétrica (rango 14.4–31.9 °C, sin outliers extremos observados).

b) **Dispersión de `temperatura_c`:** la **desviación estándar** (≈2.8 °C) es la medida adecuada.
   Nos dice que el proceso de negocio subyacente (temperatura ambiente en Medellín durante el
   período de marzo observado) tiene una variabilidad moderada y estable — consistente con el clima
   templado de la ciudad, sin cambios bruscos día a día que obliguen a un monitoreo climático más
   granular que el actual.

c) **Escenario donde el IQR sería más informativo que la desviación estándar:** si apareciera un
   grupo de lecturas de temperatura con errores de captura extremos (ej. un sensor reportando -40 °C
   por una falla), la desviación estándar se vería arrastrada por esos valores, mientras que el IQR
   (rango entre percentil 25 y 75) seguiría reflejando la variabilidad real del 50% central de los
   datos, ignorando el ruido de captura.

### 1.3 — Cualitativo vs. cuantitativo

Variable categórica elegida: **`condicion_clima`**.

- **Resumen numérico:** tabla de frecuencias/proporciones — Soleado ≈49%, Nublado ≈30%, Lluvia ≈15%,
  sin dato ≈6% (sobre el archivo ya limpio del equipo).
- **Resumen gráfico:** gráfico de barras de frecuencia por categoría.
- **Decisión de negocio que se apoyaría en este resumen:** si la gran mayoría de las lecturas ocurren
  en condición "Soleado", la Secretaría de Movilidad puede decidir que el piloto de semaforización
  inteligente **no necesita** una lógica especial para lluvia como prioridad inicial (bajo volumen de
  observaciones de lluvia en el período), y enfocar el piloto en el patrón horario en vez del climático.

### 1.4 — Forma de la distribución y atípicos

La variable cuantitativa principal del negocio es `conteo_vehiculos`. Sin necesidad de graficarla,
esperaríamos una distribución **sesgada a la derecha, con cola larga hacia valores altos**: el
tráfico urbano tiene muchas horas de flujo bajo (madrugada) y pocas horas de flujo muy alto (pico
matutino/vespertino), un patrón típico de procesos de conteo de eventos (similar a un proceso tipo
Poisson) más que a una campana simétrica.

**Criterio para decidir "atípico pero real" vs. "error de captura":** comparar el valor contra el
rango físicamente posible dado el contexto (tipo de vía, hora del día). Un conteo de 45-50 vehículos
en hora pico en una vía Troncal es alto pero plausible; un conteo negativo o de 99999 no tiene
interpretación física posible bajo ningún contexto — es evidencia de error de sensor o de captura,
no de un evento de tráfico real por más extremo que sea.

### 1.5 — Pregunta transversal

Dos datasets con la misma media pueden requerir decisiones de negocio completamente distintas porque
la media no dice nada sobre la **forma** de la distribución (dispersión, asimetría, presencia de
outliers). En nuestro dataset, `conteo_vehiculos` tiene media 22.9 pero mediana 18: un dataset
hipotético con la misma media 22.9 pero perfectamente simétrico y de baja varianza implicaría tráfico
estable todo el día, y la recomendación sería "no se necesita intervención horaria diferenciada". El
dataset real, en cambio, tiene esa misma media pero producida por una mezcla de valles muy bajos
(≈14) y picos muy altos (≈40), lo que exige una política diferenciada por franja horaria — la media
sola oculta esta diferencia y llevaría a una decisión equivocada si se usara sola.

---

## Parte 2 — Análisis y transformación de datos con problemas (sobre el archivo CONTAMINADO)

### 2.1 — Diagnóstico de calidad (5 problemas)

1. **Valores faltantes** en `conteo_vehiculos`, `temperatura_c`, `condicion_clima` → detectado con
   `df.isnull().sum()`.
2. **Duplicados exactos de fila** → detectado con `df.duplicated().sum()` (7 pares encontrados).
3. **Formatos de fecha/hora mixtos** en `timestamp` (ISO, ISO con `Z`/UTC, epoch unix,
   `DD/MM/YYYY HH:MM`, texto largo en español) → detectado inspeccionando `df["timestamp"].unique()`
   con expresiones regulares, y contando fallos de `pd.to_datetime(df["timestamp"], errors="coerce")`.
4. **Etiquetas categóricas inconsistentes** en `condicion_clima` (`SOLEADO`/`soleado`/`Sol`, etc.) →
   detectado con `df["condicion_clima"].unique()`.
5. **Coordenadas geográficas inválidas** (lat/lon invertidas en el sensor SEN04, 121 filas) →
   detectado comparando `lat`/`lon` contra el rango geográfico válido de Medellín.

### 2.2 — Fechas

```python
df["timestamp"] = pd.to_datetime(df["timestamp"], errors="coerce", format="mixed")
```

Esta línea por sí sola **no es suficiente** en este dataset: `format="mixed"` de pandas asume la
convención `MM/DD/YYYY` para cadenas ambiguas como `04/03/2025`, y las interpretaría como **4 de
abril** en vez de **4 de marzo** — es decir, produciría una fecha *incorrecta*, no un nulo. Por eso
el equipo implementó un parser explícito por formato en el notebook (ver Tarea 3.1) que sí garantiza
0% de fallos reales.

**Si el 8% de las fechas no lograran convertirse:** las dejaríamos como **nulo explícito**, no las
eliminaríamos ni las imputaríamos. No las eliminamos porque el resto de columnas de esa fila (conteo,
clima, ubicación) sigue siendo información válida para el análisis agregado por corredor. No las
imputamos con una fecha arbitraria porque fabricaría un momento temporal que no ocurrió realmente,
lo cual es peor para un análisis por hora del día que simplemente no saberlo.

### 2.3 — Variable lógica / categórica

Variable con inconsistencias de texto: **`condicion_clima`**.

```python
df["condicion_clima"] = df["condicion_clima"].str.strip().str.lower().map(
    {"soleado": "Soleado", "sol": "Soleado",
     "nublado": "Nublado", "nubes": "Nublado",
     "lluvia": "Lluvia", "lluvioso": "Lluvia"}
).fillna("Sin dato")
```

**Criterio para decidir qué categorías son "la misma":** equivalencia semántica del fenómeno
climático descrito, no de la palabra exacta. `Sol` y `Soleado` describen el mismo estado del cielo;
`Nubes` y `Nublado`, igual; `lluvioso` y `Lluvia`, igual. Las diferencias de mayúsculas/espacios no
aportan información nueva, solo ruido de captura.

### 2.4 — Georreferenciación

**Regla de validación** (rango aproximado del área urbana de Medellín): latitud entre 6.15 y 6.35,
longitud entre -75.65 y -75.48.

```python
invalida = ~df["lat"].between(6.15, 6.35) | ~df["lon"].between(-75.65, -75.48)
```

**Discusión — ¿corregir automáticamente (swap) o marcar para revisión manual?** El equipo decidió
**no** hacer swap automático ciego. En su lugar, cada fila inválida se reemplaza por la **mediana de
coordenadas válidas del mismo `sensor_id`** (un sensor físico no cambia de ubicación, así que su
propia mediana histórica es una estimación confiable), y se deja registrada en una columna
`geo_corregida` para trazabilidad. **Riesgo del swap automático:** asume que *todo* error de
coordenada es un intercambio lat/lon simple, cuando podría ser un error distinto (truncamiento, deriva
de GPS) y el swap produciría una coordenada igualmente incorrecta con más confianza aparente.
**Riesgo de la revisión manual pura:** no escala si hay muchos sensores/filas afectadas y retrasa el
análisis sin garantía de que la revisión humana sea más precisa que la mediana histórica del propio
sensor.

### 2.5 — Imputación y valores imposibles

Variable elegida: **`conteo_vehiculos`** (valores negativos y el valor centinela `99999`).

a) **Criterio para el rango válido:** `[0, 500]`. El límite inferior es físico (no existen conteos
   negativos de vehículos). El límite superior es conservador: el archivo limpio de referencia
   muestra un máximo real de ~54 vehículos por lectura de 2 horas en el período observado, así que
   500 no descarta ningún pico de tráfico genuino, pero sí captura inequívocamente el valor
   centinela `99999`.
b) **Estrategia:** tratar los valores fuera de rango como nulos, y luego imputar (junto con los
   nulos originales) con la **mediana del mismo sensor a esa misma hora del día** — no la media
   global, para no aplanar el patrón horario real.
c) **Sesgo si la estrategia fuera incorrecta:** si se imputara con la media global (ignorando sensor
   y hora), los picos de tráfico reales de 6-8h y 16-18h se subestimarían y las horas de valle se
   sobrestimarían, aplanando artificialmente exactamente el patrón horario que sustenta la
   recomendación de negocio de este taller (franjas horarias para el piloto de semaforización).

### 2.6 — Duplicados y llave de negocio

**Llave de negocio:** `sensor_id` + `timestamp`. Cada sensor reporta exactamente una lectura por
franja de 2 horas (confirmado en el archivo limpio: 240 lecturas por sensor en 20 días); dos filas
con el mismo sensor y el mismo instante no pueden representar dos eventos de tráfico distintos, son
la misma medición duplicada en el pipeline de ingestión.

**Por qué un duplicado exacto de fila no es lo mismo que un duplicado de evento de negocio:** un
duplicado exacto es un caso particular donde *además* todas las demás columnas coinciden. En este
dataset encontramos 7 pares adicionales que comparten `sensor_id`+`timestamp` pero difieren en otra
columna (ej. `condicion_clima` nulo en una copia y "Soleado" en la otra, o `lat`/`lon` intercambiados
entre las dos copias) — `duplicated()` sin especificar `subset` no los detecta, y si no se resuelven,
un análisis por hora terminaría promediando dos versiones contradictorias del mismo instante real.

---

## Parte 3 — Interpretación de resultados y decisión

### 3.C — Decisión de gestión de tráfico (Dataset C: Movilidad urbana)

**a) Picos de tráfico e hipótesis del fenómeno urbano:**
La gráfica por hora muestra dos picos claros: **6:00-8:00** y **16:00-18:00**, con conteos
promedio de ~39-41 vehículos frente a ~14-15 en horas valle (casi el triple). La hipótesis de negocio
más simple y consistente con el patrón es el **horario de entrada y salida laboral/escolar típico de
una ciudad como Medellín**: el pico matutino corresponde al desplazamiento hacia trabajo/estudio y el
vespertino al regreso a casa.

**b) ¿El tipo de vía no es relevante?**
La tabla muestra conteos promedio casi idénticos entre Arteria (22.5), Local (23.0) y Troncal (23.2).
Al revisar la gráfica por sensor individual, encontramos que **tampoco allí hay diferencias
importantes** (rango 22.3–23.3 vehículos entre los 6 sensores) — es decir, la evidencia por sensor
**no matiza ni contradice** la conclusión de la tabla, sino que la **refuerza**: ni el tipo de vía ni
el corredor individual explican por sí solos la variación de tráfico observada en este período. La
señal estadística fuerte está en la **hora del día** (variación de ~3x entre pico y valle), no en
dónde está ubicado el sensor. Esto es un hallazgo honesto aunque no sea el que la pregunta parecía
anticipar: los datos no sostienen priorizar por tipo de vía.

**c) Recomendación final:**
Iniciar el piloto de semaforización inteligente en **las franjas 6:00-8:00 y 16:00-18:00, en los
seis corredores monitoreados por igual**, en vez de concentrar el esfuerzo en un solo tipo de vía —
los datos no justifican esa priorización. Si el presupuesto solo alcanza para una fase inicial,
priorizar los sensores SEN04 y SEN06 (los de mayor conteo promedio). La variable adicional que
monitorearíamos para validar el impacto del piloto es la **condición climática** (`condicion_clima`),
en particular los días de lluvia: aunque en este período su efecto promedio sobre el nivel de tráfico
fue débil, la cobertura de datos de clima integrados fue baja (12% de las lecturas), así que no se
puede descartar un efecto real que el piloto debería seguir midiendo.
