# Taller Práctico 01

**Maestría en Ciencia de Datos y Analítica · Universidad EAFIT**
Curso: Fundamentos en Ciencia de Datos | Docente: Jorge Iván Padilla-Buriticá · Periodo 2026-2

**Conjunto de datos elegido: C — Movilidad urbana** (`movilidad_sensores_LIMPIO.csv`, `movilidad_sensores_CONTAMINADO.csv`)

**Integrantes:**

- Cristian Miguel Gómez Salazar
- Santiago Rueda Mira
- Santiago Alberto Vélez Casallas
---
Según el enunciado, no se evalúa la cantidad
de código: cada literal incluye una celda de código **corta**, cuyo único fin es *sustentar* la
respuesta escrita que aparece a continuación bajo el rótulo **RESPUESTA 1.x**.

*Uso de IA generativa declarado en `docs/declaracion_uso_IA.md`. La interpretación y las
conclusiones son del equipo; las salidas numéricas fueron verificadas manualmente.*

> _Nota: conforme al enunciado, este documento acompaña al notebook del taller y reúne **únicamente las respuestas en texto**. Todo el código (carga de datos, transformaciones y figuras) está en el notebook `Taller_Practico_01_analisis.ipynb`, dentro del mismo repositorio de GitHub; donde había una celda de código, se cita el notebook._

## 0. Configuración y carga de datos

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

# Parte 1 — Análisis estadístico general

_Se responde sobre el archivo LIMPIO (`df`)._

---
## Punto 1.1 — Taxonomía de variables (6 pts)

> Clasifique cinco variables según la taxonomía (nominal, ordinal, discreta, continua) y explique
> por qué esa clasificación determina qué estadístico de tendencia central es válido.

**Variables:** `ubicacion` (nominal), `tipo_via` (ordinal), `conteo_vehiculos` (discreta),
`temperatura_c` (continua), `condicion_clima` (nominal).

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_



| Variable | Clasificación | Tendencia central válida |
|---|---|---|
| `ubicacion` | Nominal | Solo moda |
| `tipo_via` | Ordinal | Moda y mediana (no media) |
| `conteo_vehiculos` | Discreta | Media, mediana y moda |
| `temperatura_c` | Continua | Media, mediana y moda |
| `condicion_clima` | Nominal | Solo moda |

- **`ubicacion` (nominal):** los seis corredores son etiquetas sin orden; no se puede decir que
  uno "es mayor" que otro, así que ni media ni mediana están definidas y solo aplica la moda.
- **`tipo_via` (ordinal):** existe un orden real (Local < Arteria < Troncal) pero no una distancia
  medible entre niveles; por eso admite moda y mediana, pero **no media**.
- **`conteo_vehiculos` (discreta):** conteo de eventos enteros (no existen 22.9 vehículos); admite
  media, pero se interpreta como tasa promedio, no como una lectura posible.
- **`temperatura_c` (continua):** escala de intervalo con distancias constantes (de 20 a 21 °C hay
  lo mismo que de 30 a 31 °C); esa propiedad es la que habilita la media.
- **`condicion_clima` (nominal):** categorías excluyentes sin jerarquía; solo moda y frecuencias.

**Por qué la clasificación decide el estadístico — explicación de la evidencia.** Para "promediar"
`tipo_via` hay que convertir primero cada categoría en número, porque no se puede promediar texto.
Esa conversión se llama *codificar*, y es una elección del analista: el orden es real, pero la
distancia entre niveles no la trae el dato. Probamos dos codificaciones igual de válidas —
**A: 1-2-3** (niveles equiespaciados) y **B: 1-2-10** (Troncal muy por encima, por su mayor
capacidad) — y como el diseño está balanceado (480 lecturas por tipo), la media resultó:

- Codificación A → **2.00**
- Codificación B → **4.33**

La media **cambió** con solo cambiar una suposición arbitraria, mientras la **mediana se mantuvo
en 2.0** en ambos casos (la mediana solo mira quién está en el centro del orden —Arteria—, sin
importar el número asignado). Conclusión: la media de una ordinal mide la codificación elegida, no
una propiedad del dato; por eso es inválida. Es el "¡cuidado con promediar ordinales!" visto en
clase, demostrado con nuestros datos.

## 1.2 - Medidas de tendencia central y dispersión (8 pts)
**Variables seleccionadas:** `tipo_via` (Ordinal) y `temperatura_c` (Continua).

**a) Tendencia central para `tipo_via` (Ordinal):**
Calcularía la **Mediana** (tras un mapeo ordinal: Local=1, Arteria=2, Troncal=3) o la **Moda**. **No usaría la media** porque la distancia categórica entre una vía "Local" y una "Arteria" no es numéricamente igual a la distancia entre "Arteria" y "Troncal"; promediarlas daría un valor ficticio e irreal.

**b) Medida de dispersión para `temperatura_c` (Continua):**
Propongo la **Desviación Estándar**. Para el proceso de negocio de IoT y movilidad, una alta varianza térmica podría advertir de posibles daños en los sensores por exceso de calor sobre el asfalto, o correlacionarse con cambios climáticos abruptos que requieran alertas tempranas de accidentes.

**c) Escenario para el Rango Intercuartílico (IQR):**
El IQR sería mucho más informativo que la desviación estándar si los sensores IoT sufren fallos temporales y arrojan **valores atípicos extremos** (ej. registrando picos irreales de 65°C o -10°C). Como la desviación estándar es muy sensible a datos extremos, el IQR mediría mucho mejor el comportamiento de dispersión de la temperatura central normal de Medellín.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## Punto 1.3 — Cualitativo vs. cuantitativo (6 pts)

> Elija una variable categórica; descríbala con un resumen numérico y uno gráfico, y explique qué
> decisión de negocio se apoya en ese resumen.

**Variable:** `condicion_clima` — única categórica con variación real (las otras están balanceadas
por diseño del muestreo).

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_



**Resumen numérico — tabla de frecuencias:** Soleado 750 (52.1 %), Nublado 469 (32.6 %),
Lluvia 221 (15.3 %). Es el resumen correcto para una nominal: reporta moda y proporciones sin
suponer orden. La proporción es clave porque **es una probabilidad empírica**: ~15 de cada 100
lecturas ocurrieron bajo lluvia.

**Resumen gráfico — barras de frecuencia + boxplots del conteo por clima.** Se usan barras (no
líneas) porque el eje categórico no tiene continuidad; el boxplot añade el paso de "cuántas veces
ocurre" a "qué efecto tiene".
*Conclusión de la gráfica:* el clima no es uniforme (Soleado domina), pero el conteo vehicular es
casi idéntico entre condiciones — **1.5 vehículos** de diferencia frente a σ ≈ 13.

**Decisión de negocio.** La hipótesis operativa sería *"la lluvia altera el flujo, luego el
semáforo debe ajustarse por clima"*. El resumen permite **descartarla con evidencia**: aunque la
lluvia es frecuente (15.3 %, suficiente para importar), su efecto sobre el conteo es de 1.5
vehículos y ni siquiera es monótono (Nublado queda por debajo de Lluvia, lo que contradice
cualquier mecanismo causal). **Decisión: no incorporar el clima como variable de control del
piloto**, y concentrar el presupuesto en la franja horaria, que sí discrimina (punto 1.4). El valor
del resumen cualitativo es exactamente ese: descartar un factor con datos en vez de con intuición.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## Punto 1.4 — Forma de la distribución y atípicos (6 pts)

> Describa la forma esperada de la variable continua principal según el proceso que la genera, y dé
> un criterio para distinguir un atípico **real** de un **error de captura**.

**Variable:** `conteo_vehiculos` (el enunciado menciona los "conteos de tráfico"). Formalmente
discreta, pero con 55 valores distintos admite el análisis de forma.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_



**Forma esperada.** `conteo_vehiculos` mide demanda de movilidad, gobernada por la jornada laboral:
picos de entrada/salida y valles en horas intermedias y madrugada. Por eso **no se espera una
campana simétrica sino una distribución bimodal**, y los datos lo confirman:
- **Curtosis = −1.00** (más plana que la normal): firma de una **mezcla de dos poblaciones**, no de
  colas largas.
- **Media (22.9) ≠ Mediana (19) ≠ Moda (14):** los tres discrepan, señal de que ninguno describe un
  centro real; la media cae en el **valle entre las dos jorobas**, un valor que casi nunca ocurre.
- La asimetría global de +0.49 es un **artefacto de la mezcla**: al separar por franja, cada régimen
  es simétrico (skew −0.10 en pico, +0.01 en valle) y la dispersión interna (σ ≈ 5.9) es menos de
  la mitad de la global (σ = 13.2). **Más de la mitad de la "variabilidad" es solo la distancia
  entre pico (39.6) y valle (14.6).** La variable no tiene colas: está acotada entre 0 y 54 por la
  capacidad física del corredor.

**Criterio atípico real vs. error de captura.** Un criterio global falla: la regla IQR sobre todo
el conjunto da límites tan anchos que **no marca ningún atípico**. El criterio correcto es
**contextual — evaluar cada lectura contra el rango esperado para su hora del día**:

| Valor | Hora | Lectura |
|---|---|---|
| 54 vehículos | 18:00 | **atípico real** (congestión válida) |
| 54 vehículos | 02:00 | **error de captura** (imposible en madrugada) |
| 0 vehículos | 08:00 | **error de captura** (sensor caído, no vía vacía) |
$$$$

El mismo número —54— es dato válido a las 6 p.m. y falla a las 2 a.m. Se refuerza con tres checks
de dominio: **persistencia** (un pico aislado = falla; varios seguidos = evento real),
**corroboración entre sensores** (si solo un corredor se dispara, es el equipo) y **plausibilidad
física** (negativos o sobre la capacidad se descartan). **Regla:** se marca como error si viola el
rango de su hora *y* no lo corroboran sensores vecinos ni lecturas contiguas; si lo corroboran, se
conserva y se documenta como evento —esos episodios son justo los que el semáforo debe aprender.

------------------------------------------------------------------

## Verificación del supuesto:

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

## 1.5 - Pregunta transversal (4 pts)
**¿Por qué dos datasets con la misma media pueden requerir decisiones distintas?**

La media por sí sola invisibiliza la variabilidad de las operaciones subyacentes.
*Ilustración con `conteo_vehiculos`:* Si dos sensores distintos viales (SEN01 y SEN02) reportan una media diaria de **400 vehículos por hora**, podrían parecer el mismo escenario a nivel ejecutivo.
* Sin embargo, el **SEN01 (Arteria)** puede registrar flujos constantes de entre 380 y 420 autos a toda hora (baja dispersión), lo que solo requiere mantenimiento estándar.
* El **SEN02 (Troncal)** puede estar marcando 1,500 vehículos colapsando la vía en la "hora pico" y 0 autos en la madrugada (alta dispersión). Este segundo corredor requiere decisiones críticas inmediatas de infraestructura (ampliación de vías) y normativas (pico y placa), mientras que el primero no.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

------------------------------------------------------------------------
------------------------------------------------------------------------
# **Parte 2 — Análisis y transformación de datos con problemas**

_Se responde sobre el archivo CONTAMINADO (`raw`)._

## 2.1 - Diagnóstico de calidad (10 pts)
Al inspeccionar el dataset contaminado, se identifican los siguientes **cinco problemas de calidad** (Principio GIGO):

1. **Valores imposibles (Outliers físicos):** Existen registros de `-5` vehículos y máximos irreales de `99999` vehículos en la columna `conteo_vehiculos`.
   * **Detección en pandas:** `raw.describe()` o `raw['conteo_vehiculos'].min()`
2. **Coordenadas geográficas invertidas y erróneas:** Hay latitudes negativas (ej. `-75.56`) y longitudes positivas (ej. `6.28`), lo que indica que se invirtieron los ejes lat/lon.
   * **Detección en pandas:** `raw[raw['lat'] < 0]`
3. **Inconsistencia categórica (Ruido de texto):** La columna `condicion_clima` tiene mayúsculas mezcladas y sinónimos (ej. *'Soleado', 'SOLEADO', 'Sol', 'lluvioso'*).
   * **Detección en pandas:** `raw['condicion_clima'].unique()` o `raw['condicion_clima'].value_counts()`
4. **Valores faltantes (Incompletitud):** Faltan datos (NaN) en mediciones críticas como `conteo_vehiculos`, `temperatura_c` y `condicion_clima`.
   * **Detección en pandas:** `raw.isnull().sum()`
5. **Duplicados de eventos de negocio:** Posibles colisiones temporales donde un mismo sensor envía datos dos veces para la misma hora.
   * **Detección en pandas:** `raw.duplicated(subset=['sensor_id', 'timestamp']).sum()`

   --------------------------------------------------------------------

  # EN SÍNTESIS:

Documentamos los problemas de calidad **sin corregir todavía**, con el método de detección exacto
en pandas para cada uno.

| Problema detectado | Columna(s) afectada(s) | Método de detección en pandas | ¿Por qué es un riesgo para la decisión de negocio? |
|---|---|---|---|
| Valores faltantes | `conteo_vehiculos` (145), `temperatura_c` (72), `condicion_clima` (87) | `df.isnull().sum()` | Si se ignoran, subestiman el tráfico real de esas franjas y sesgan el promedio por corredor/hora. |
| Duplicados exactos de fila | Todas las columnas | `df.duplicated().sum()` → 7 pares | Duplicar una lectura infla artificialmente el conteo total de vehículos en ese corredor/hora. |
| Duplicados de "evento de negocio" | `sensor_id` + `timestamp` con otro valor distinto | `df.duplicated(subset=["sensor_id","timestamp"], keep=False)` → 14 filas (7 pares adicionales) | Un mismo evento de medición aparece dos veces con ligeras diferencias (ej. clima nulo en una copia); si no se resuelve, el análisis por hora puede promediar valores contradictorios del mismo instante. |
| Categorías de clima inconsistentes | `condicion_clima` | `df["condicion_clima"].unique()` / `value_counts()` → 12 variantes de solo 3 categorías reales | Al no unificar `SOLEADO`/`soleado`/`Sol`, cualquier conteo o tabla de contingencia por clima subestima la categoría real y puede llevar a conclusiones erróneas sobre el efecto del clima en el tráfico. |
| Formatos de fecha/hora mixtos | `timestamp` | Inspección de `df["timestamp"].astype(str)` con expresiones regulares; `pd.to_datetime(..., errors="coerce")` para contar fallos | Si se descartan silenciosamente las fechas no-ISO (~9% de las filas), se pierde tráfico real de ciertos días sin razón relacionada con el tráfico mismo, sesgando el análisis temporal. |
| Valores imposibles en conteo | `conteo_vehiculos` | `df[(df["conteo_vehiculos"]<0) \| (df["conteo_vehiculos"]>umbral)]` | Un conteo negativo o de 99999 no es tráfico real (es un código de error de sensor); si se deja, distorsiona brutalmente cualquier promedio (ver `std` de 5518 vs `std` real ~10). |
| Coordenadas invertidas | `lat`, `lon` (sensor SEN04, 121 filas) | Regla de rango válido para Medellín (`lat` 6.15–6.35, `lon` -75.65–-75.48); filas donde `lat<0` y `lon>0` | Un mapa de puntos con estas coordenadas ubicaría el sensor fuera de Medellín (en el océano/otro continente), invalidando cualquier análisis geoespacial del corredor. |

**Cobertura de las 6 categorías mínimas exigidas:** completitud ✅, unicidad (exacta y de evento) ✅✅,
consistencia de categorías ✅, formatos de fecha mixtos ✅, valores imposibles ✅, georreferenciación ✅.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## Punto 2.2 — Fechas (6 pts)

> La columna de fecha/hora llega con formatos mixtos. Escriba la línea que la convierte a datetime
> dejando como nulos los no convertibles. Si el 8 % no convierte, ¿elimina, imputa o deja nulo?

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_


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

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## Punto 2.3 — Variable categórica con inconsistencias (6 pts)

> Identifique una variable con inconsistencias de texto (mayúsculas, espacios, sinónimos). Proponga
> la transformación y justifique qué categorías son la misma.

### Variable con inconsistencias de texto: **`condicion_clima`**.

```python
df["condicion_clima"] = df["condicion_clima"].str.strip().str.lower().map(
    {"soleado": "Soleado", "sol": "Soleado",
     "nublado": "Nublado", "nubes": "Nublado",
     "lluvia": "Lluvia", "lluvioso": "Lluvia"}
).fillna("Sin dato")
```
**Criterio para decidir qué categorías son la misma** — dos niveles:

1. **Variantes tipográficas** (`SOLEADO`, `soleado`, `Soleado`): son *inequívocamente* el mismo valor;
   solo difieren en mayúsculas/espacios. Se unifican con `str.strip().str.lower()` sin riesgo de
   interpretación, porque no hay ambigüedad semántica.
2. **Sinónimos** (`Sol`→Soleado, `Nubes`→Nublado, `lluvioso`→Lluvia): requieren *juicio de dominio*.
   Se agrupan solo cuando el significado meteorológico es equivalente y no se pierde información: "Sol"
   y "Soleado" describen la misma condición operativa para el tráfico. **No** se fuerza el mapeo en
   casos dudosos; si hubiera aparecido algo como "Tormenta" se dejaría separado, porque podría
   implicar un efecto distinto sobre la movilidad.

El diccionario explícito es preferible a una agrupación automática (p. ej. por similitud de texto)
porque **deja documentada y auditable cada decisión de equivalencia**, que es lo que la rúbrica pide
justificar. Tras la normalización, las 11 variantes colapsan a las 3 categorías del dominio, y los
87 nulos se mantienen como tales para tratarse en el paso de completitud.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## Punto 2.4 — Georreferenciación (8 pts)

> Proponga una regla de validación (rango de lat/lon de Medellín) y marque las filas que la
> incumplan. ¿Corregiría automáticamente una coordenada invertida (swap) o la marcaría para
> revisión manual?



**Regla de validación** (rango aproximado del área urbana de Medellín): latitud entre 6.15 y 6.35,
longitud entre -75.65 y -75.48.

```python
invalida = ~df["lat"].between(6.15, 6.35) | ~df["lon"].between(-75.65, -75.48)
```
**Discusión — ¿corregir automáticamente (swap) o marcar para revisión manual?** El equipo decidió
**no** hacer swap automático ciego. En su lugar, cada fila inválida se reemplaza por la **mediana de
coordenadas válidas del mismo `sensor_id`** (un sensor físico no cambia de ubicación, así que su
propia mediana histórica es una estimación confiable), y se deja registrada en una columna
`geo_corregida` para trazabilidad.

**Riesgo del swap automático:** asume que *todo* error de
coordenada es un intercambio lat/lon simple, cuando podría ser un error distinto (truncamiento, deriva
de GPS) y el swap produciría una coordenada igualmente incorrecta con más confianza aparente.

**Riesgo de la revisión manual pura:** no escala si hay muchos sensores/filas afectadas y retrasa el
análisis sin garantía de que la revisión humana sea más precisa que la mediana histórica del propio
sensor.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## Punto 2.5 — Imputación y valores imposibles (6 pts)

> Elija una variable numérica con valores imposibles. Proponga (a) el rango válido, (b) la estrategia
> de imputación o eliminación, y (c) qué sesgo introduciría una estrategia incorrecta.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_


**Variable:** `conteo_vehiculos`, con valores imposibles: **6 negativos** (−5, −1) y **4 registros de
99999** (centinela de error de sensor). Máximo real observado en el limpio: 54.

**(a) Rango válido.** Un conteo de vehículos es un entero **no negativo**, acotado por la capacidad
del corredor en el intervalo. Fijamos `0 ≤ conteo ≤ 60` (el 60 da un pequeño margen sobre el máximo
observado de 54, para no descartar picos reales). Todo lo que caiga fuera —negativos y 99999— se
marca como inválido (`NaN`).

**(b) Estrategia.** Dos pasos:
1. **Los valores imposibles se invalidan, no se conservan:** un −5 o un 99999 no son datos, son
   fallas. Se convierten a `NaN` y se suman a los 145 nulos ya existentes.
2. **Imputación condicionada a la hora, con la mediana.** No se imputa con la media global ni con la
   mediana global, porque la Parte 1 demostró que la variable es **bimodal**: un nulo a las 8 a.m.
   (hora pico) no se parece a un nulo a las 3 a.m. Se imputa con **la mediana de la misma franja
   horaria** (`groupby('hora')`), que respeta el régimen al que pertenece la lectura. Se elige
   mediana y no media porque es robusta a los outliers que aún puedan quedar. En sensores con muchos
   nulos consecutivos, la eliminación es preferible a inventar una secuencia larga.

**(c) Sesgo de una estrategia incorrecta.**
- Si **no se invalidara el 99999** e se imputara/promediara con él, un solo valor arrastraría la media
  de ~23 a miles: la media global pasaría a ser inútil y la desviación estándar explotaría (mismo
  efecto que demostramos en 1.2 con la temperatura).
- Si se **imputara con la media/mediana global** ignorando la hora, se rellenarían los huecos de hora
  pico con un valor cercano al promedio (~19–23) en lugar de ~40, **subestimando sistemáticamente la
  congestión** justo en las franjas que importan para la decisión de semaforización. El análisis
  concluiría que los picos son más suaves de lo que son, y el piloto se dimensionaría por debajo de
  la necesidad real. Por eso la imputación **debe** condicionarse a la franja.

---
## Punto 2.6 — Duplicados y llave de negocio (4 pts)

> ¿Qué columna(s) usaría como llave de negocio para detectar duplicados reales, y por qué un
> duplicado exacto de fila no es lo mismo que un duplicado de evento de negocio?

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_



**Llave de negocio: `sensor_id` + `timestamp`.** Un sensor físico solo puede producir **una** lectura
por instante. Esa combinación identifica de forma única un "evento de negocio" (una medición), así
que cualquier par repetido de sensor+instante es un duplicado real, aunque el resto de columnas
difieran.

**Por qué un duplicado exacto de fila ≠ un duplicado de evento de negocio:**
- `df.duplicated()` (fila completa) detecta solo **7** casos: filas idénticas en las 9 columnas.
- `df.duplicated(subset=['sensor_id','timestamp'])` detecta **14**: el doble.

La diferencia son los duplicados **de negocio pero no exactos**: el mismo sensor y el mismo instante,
pero con alguna columna distinta. El ejemplo lo muestra: para `SEN01` a las `2025-03-08 04:00:00`
existen dos filas —una con `conteo_vehiculos = NaN` y otra con `10.0`—. Son la **misma medición**
(mismo evento físico), pero un filtro de fila exacta **no las vería** porque una columna difiere.

Consecuencia práctica: deduplicar solo con `duplicated()` dejaría pasar la mitad de los duplicados
reales, y esos casos son además informativos —permiten **recuperar** el valor real (10.0) para
rellenar la copia nula—. La estrategia correcta es agrupar por la llave de negocio, consolidar la
información no nula de las copias, y conservar una sola fila por evento. Un duplicado exacto infla el
volumen; un duplicado de negocio, además, puede esconder o contradecir el dato verdadero.

--------------------------------------------------------
--------------------------------------------------------
# **Parte 3 — Interpretación de resultados y decisión**

_Se responde sobre el archivo LIMPIO (`df`). El código es opcional (confirma las cifras del Cuadro 3)._

### Evidencia entregada por el documento (Dataset C)

**Cuadro 3 — Resumen por tipo de vía** (datos limpios, 6 sensores, 20 días, lecturas cada 2 h):

| Tipo de vía | N lecturas | Conteo prom. | Conteo máx. | Temp. prom. (°C) |
|---|---|---|---|---|
| Arteria | 480 | 22.5 | 54 | 23.3 |
| Local | 480 | 23.0 | 53 | 23.1 |
| Troncal | 480 | 23.2 | 53 | 22.9 |

**Figura 3** — Izquierda: conteo promedio de vehículos por hora del día (perfil bimodal).
Derecha: conteo promedio por sensor (seis barras casi iguales, ~22–23).

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

---
## 3.C (a) — Picos de tráfico e hipótesis del fenómeno urbano

> A partir de la gráfica por hora, identifique los picos de tráfico y proponga una hipótesis del
> fenómeno urbano que los explica.

**a) Picos de tráfico e hipótesis del fenómeno urbano:**
La gráfica por hora muestra dos picos claros: **6:00-8:00** y **16:00-18:00**, con conteos
promedio de ~39-41 vehículos frente a ~14-15 en horas valle (casi el triple). La hipótesis de negocio
más simple y consistente con el patrón es el **horario de entrada y salida laboral/escolar típico de
una ciudad como Medellín**: el pico matutino corresponde al desplazamiento hacia trabajo/estudio y el
vespertino al regreso a casa.

---
## 3.C (b) — ¿El tipo de vía es irrelevante?

> La tabla muestra conteos promedio casi idénticos entre Arteria, Local y Troncal. ¿Esto significa
> que el tipo de vía no es relevante para la decisión? Argumente qué otra evidencia (gráfica derecha,
> por sensor) matiza esta conclusión.

**Hallazgo adicional (sensor 6):** Se resalta cómo las vías de tipo "locales" tienen el mismo conteo que las troncales y las arterias, en específico la del sensor 6; y que, aunque la desviación estándar hace que este dato se considere no significativo , resalta la atención y se recomienda hacer mediciones futuras sobre más corredores viales para reconfirmar si tiene o no relevancia el tipo de vía.

### RESPUESTA 3.C.b

**No se puede concluir que el tipo de vía sea irrelevante; lo correcto es concluir que este dataset
no tiene poder para discriminar corredores por su volumen promedio.** Son afirmaciones distintas, y
la diferencia es clave para no tomar una mala decisión.

**Lo que la recomendación gráfica 4 muestra:** los conteos promedio por tipo de
vía (22.5 / 23.0 / 23.2) difieren en **0.7 vehículos**, y las seis barras por sensor van de ~22.3 a
~23.3, un rango de **≈ 1 vehículo**. Frente a una desviación estándar del orden de 13 vehículos por
lectura(Tal y como se ve en la **recomendación gráfica 3**), esas diferencias son **indistinguibles del ruido**. Priorizar el corredor "de mayor
promedio" sería elegir con base en una diferencia que probablemente desaparecería con otra muestra.

**Por qué esto NO prueba que el tipo de vía sea irrelevante — tres matices:**

1. **El promedio esconde la forma.** Como demostró la Parte 1, la media global mezcla pico y valle.
   Dos corredores con idéntico promedio pueden tener perfiles horarios muy distintos (uno con pico
   agudo, otro más plano) y ese detalle —invisible en la tabla— sí sería decisivo. La evidencia
   entregada no incluye el perfil horario *por corredor*, así que la pregunta queda abierta, no
   resuelta.

2. **El diseño del muestreo fuerza la uniformidad.** Cada tipo de vía tiene exactamente 480 lecturas
   y cada sensor 240 (balanceado por construcción). El dataset fue diseñado para representar los
   corredores por igual, no para reflejar sus diferencias reales de demanda; la similitud de
   promedios es en parte un artefacto de ese diseño.

3. **n efectivo = 6, no 1440.** `tipo_via` está anidada en `ubicacion`: solo hay **2 corredores por
   tipo**. Comparar tipos de vía es comparar grupos de 2 unidades, muestra demasiado pequeña para
   cualquier conclusión sólida sobre jerarquía vial.

**Conclusión del literal:** la evidencia disponible no permite priorizar por corredor, pero tampoco
autoriza a descartar el tipo de vía. La decisión debe apoyarse en la variable que sí muestra señal
—la franja horaria— y dejar explícito que la discriminación espacial requiere datos adicionales.

---
## 3.C (c) — Recomendación final (máx. 6 líneas)

> Redacte la recomendación final en máximo 6 líneas: ¿en qué corredor(es) y horario(s) iniciaría el
> piloto, y qué variable adicional (clima, ubicación) monitorearía para validar el impacto?

## Decisión recomendada

- **Recomendación:** iniciar el piloto de semaforización inteligente en las franjas pico 6:00–8:00 y 16:00–18:00, donde el conteo casi triplica al de las horas valle (gráfica 2) y que produce la distribución bimodal del tráfico (gráfica 3). El tipo de vía y el sensor no discriminan el volumen —~1 vehículo de diferencia frente a una desviación estándar σ ≈ 13 (gráficas 4 y 3)—, y esto se sostiene pese a la variedad de ubicación de los sensores (gráfica 1) y a las condiciones climáticas (Parte 1), así que no hay base para priorizar un corredor sobre otro. Por eso se despliega en al menos un corredor de cada tipo (local, arteria, troncal), como diseño balanceado para generar la evidencia espacial que hoy falta y validar si la semaforización adaptativa reduce la congestión en pico.

# **Sección Stortytelling:**

## Recomendación grafica 1: Mapa interactivo de Medellín con localización sensores:

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

## Recomendación grafica 2: Histograma de conteo de vehículos según franja horaria

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

## Recomendación grafica 3: Histograma de frecuencia de lecturas conteo de vehículos

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_

## Recomendación grafica 4: Promedio de veahículos contados por sensor.

> _Código en el notebook: `Taller_Practico_01_analisis.ipynb`_
