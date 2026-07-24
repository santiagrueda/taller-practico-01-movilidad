# Taller Práctico 01 — [Nombre del equipo]

**Curso:** Fundamentos en Ciencia de Datos — Maestría en Ciencia de Datos y Analítica, EAFIT
**Conjunto de datos elegido:** C - Movilidad urbana
**Fecha límite de entrega:** domingo 26 de julio de 2026
**Fecha de entrega real:** [dd/mm/aaaa]

**Integrantes del equipo:**

| Nombre completo | Cédula         |
| ---------------- | -------------- |
| [Nombre 1]      | [N° de cédula] |
| [Nombre 2]      | [N° de cédula] |
| [Nombre 3]      | [N° de cédula] |

---

## 1. Resumen ejecutivo

El área de Movilidad de Medellín necesita decidir en qué corredores y horarios pilotear
semaforización inteligente. Analizamos 20 días de lecturas de 6 sensores de tráfico (cada 2 horas),
integrados con un log de clima. Tras limpiar el archivo (fechas en 5 formatos distintos, categorías
de clima inconsistentes, duplicados, conteos imposibles y coordenadas invertidas en un sensor),
encontramos que el tráfico varía casi 3 veces entre horas pico (6-8h y 16-18h) y horas valle, mientras
que el tipo de vía y el sensor individual casi no discriminan el volumen promedio. Recomendamos
iniciar el piloto en los 6 corredores por igual, concentrado en esas dos franjas horarias, y seguir
monitoreando el clima como variable de control aunque su efecto no fue concluyente con los datos
disponibles.

## 2. Pregunta de negocio

- **Pregunta ancla del conjunto de datos:** ¿En qué corredores y horarios se debe pilotear
  semaforización inteligente?
- **Pregunta específica que el equipo decidió responder:** ¿cuál es la probabilidad de que un
  corredor supere un umbral de congestión ("Alto") en una franja horaria específica, sin importar su
  tipo de vía? (ver sección "Decisión Recomendada" del notebook).

## 3. Estructura del repositorio

```
.
├── README.md
├── requirements.txt
├── data/
│   ├── raw/                  # datos originales (sin modificar)
│   └── processed/            # datos ya limpios, generados por el notebook
├── notebooks/
│   └── taller_practico_01_analisis.ipynb
├── results/
│   ├── figuras/
│   └── tabla_diagnostico_gigo.csv
├── taller_practico/
│   ├── Taller_Practico_01.tex           # enunciado original
│   ├── Taller_Practico_01.pdf           # enunciado original
│   └── Taller_Practico_01_respuestas.md # respuestas Parte 1, 2 y 3
└── docs/
    └── declaracion_uso_IA.md
```

## 4. Cómo reproducir el análisis (Solamente vía terminal)

```bash
# 1. Clonar el repositorio
git clone <url-del-repo>
cd <nombre-repo>

# 2. Crear entorno e instalar dependencias
pip install -r requirements.txt

# 3. Ejecutar el notebook de inicio a fin
jupyter nbconvert --to notebook --execute --inplace notebooks/taller_practico_01_analisis.ipynb
# o abrirlo interactivamente:
jupyter notebook notebooks/taller_practico_01_analisis.ipynb
```

**Google Colab:** una vez el equipo haga `git push` a GitHub, abran
`https://colab.research.google.com/github/<usuario>/<nombre-repo>/blob/main/notebooks/taller_practico_01_analisis.ipynb`.
La primera celda del notebook detecta automáticamente que está corriendo en Colab, clona el
repositorio y ajusta el directorio de trabajo — **solo deben reemplazar `REPO_URL` en esa celda**
por la URL real del repositorio antes de ejecutar.

## 5. Principales hallazgos

| #   | Hallazgo | Evidencia (tabla/figura) |
| --- | -------- | ------------------------ |
| 1   | El tráfico tiene dos picos claros (6-8h y 16-18h), ~3x el volumen de las horas valle | `results/figuras/01_conteo_por_hora.png` |
| 2   | El tipo de vía y el sensor individual casi no discriminan el volumen promedio de tráfico | `results/figuras/04_conteo_por_sensor.png` |
| 3   | `conteo_vehiculos` está sesgado a la derecha (media 22.9 ≠ mediana 18) | `results/figuras/02_histograma_conteo.png` |

*(Ver notebook, sección Tarea 4, para el detalle cuantitativo y cualitativo completo)*

## 6. Problemas de calidad de datos encontrados (resumen GIGO)

7 problemas documentados: valores faltantes, duplicados exactos, duplicados de evento de negocio,
categorías de clima inconsistentes, formatos de fecha mixtos (5 formatos), valores imposibles en
conteo de vehículos, y coordenadas invertidas en un sensor.

*(Tabla completa en `results/tabla_diagnostico_gigo.csv`)*

## 7. Decisión recomendada

- **Recomendación:** iniciar el piloto de semaforización inteligente en los 6 corredores monitoreados,
  concentrado en las franjas 6:00-8:00 y 16:00-18:00.
- **Costo de un Falso Positivo:** gasto de infraestructura y credibilidad del piloto sin beneficio
  medible en una franja/corredor que no lo necesitaba.
- **Costo de un Falso Negativo:** más tiempo de viaje, emisiones y accidentalidad en una franja
  crítica no intervenida; menos reversible que el Falso Positivo.
- **Limitación principal de los datos que persiste tras la limpieza:** solo 20 días de un mes y 6
  sensores; la variable de clima integrada solo cubre el 12% de las lecturas.

## 8. Declaración de uso de Inteligencia Artificial

Ver `docs/declaracion_uso_IA.md`. Resumen: se usó IA generativa (Claude Code) para sintaxis de
pandas, diagnóstico inicial de calidad de datos y un primer borrador de justificaciones; las
decisiones de criterio (estrategia de imputación, regla geográfica, llave de duplicados,
interpretación de resultados y recomendación final) fueron discutidas y validadas por el equipo.
