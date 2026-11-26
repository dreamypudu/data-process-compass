# COMPASS: perfiles de interacción del cursor y predicción de alternativas

Trabajo semestral de Machine Learning y Business Intelligence, Departamento de Ingeniería Industrial, Universidad de Concepción (2026).

**Integrantes:** Gustavo Ignacio Lara Urrutia, Catalina Andrea Alcayaga Pavez
**Profesores:** PhD Juan Carlos Caro Seguel, PhD Sebastián Rodríguez

## About

COMPASS es una simulación gamificada de decisiones de gestión en salud. El participante asume la dirección de un CESFAM y, en cada nodo de decisión, elige entre tres alternativas (A/B/C) frente a distintos personajes. Mientras decide, la plataforma registra automáticamente el movimiento del cursor sobre las alternativas.

Este repositorio se enfoca en los **datos de proceso**: lo que ocurre *antes* del clic (permanencia, cambios entre opciones y amplitud de exploración). El objetivo es superar las limitaciones del autorreporte estudiando el comportamiento observado.

### Pregunta de investigación

> ¿Qué perfiles de interacción del cursor aparecen y cuánto permite la exploración temprana anticipar la alternativa elegida?

### Usuarios beneficiados

- Tomadores de decisión en recursos humanos.
- Profesionales que buscan certificar sus habilidades.
- El equipo investigador de COMPASS, como insumo para mejorar la simulación.

### MVP

Un dashboard con el informe conductual de cada usuario. Integra las decisiones explícitas, los datos de proceso y el contexto cualitativo para mostrar cómo decide cada persona.

## Datos

Piloto COMPASS (versión CESFAM) aplicado del 3 al 19 de junio de 2026 a estudiantes de pregrado y egresados recientes de carreras de la salud.

- 60 sesiones, una por participante.
- 1.636 decisiones en 28 nodos.
- 19.153 eventos del cursor (`hover_enter` / `hover_leave`).

Fuente: respaldo PostgreSQL de COMPASS, exportado a CSV depurado. Las reglas de limpieza están en [data/README.md](data/README.md) y el diccionario de variables en [data/estructura-datos-compass.md](data/estructura-datos-compass.md).

### Tablas principales para este trabajo

| Archivo | Unidad | Uso |
|---|---|---|
| `data/process_logs.csv` | Un evento del cursor | Secuencia temporal de la exploración: tiempos desde el inicio del nodo, primera y última opción vista, número de cambios |
| `data/option_process_stats.csv` | Sesión × nodo × alternativa | Veces y tiempo total de hover por alternativa, y si fue la elegida |
| `data/explicit_decisions.csv` | Sesión × nodo | Alternativa elegida (variable objetivo), orden, día y bloque horario |
| `data/sessions.csv` | Sesión | Inicio y término de cada sesión |

El repositorio incluye además tablas con etiquetas MLQ-5X y puntajes por usuario (`mlq_labels`, `compass_moda`, `universo_alternativas`, `etiquetas_expertas_moda`, `puntajes_compass`, `puntajes_ext`). Sirven como contexto, pero no son el foco de este trabajo.

### Consideraciones de calidad

- **Dependencia.** Todos los eventos provienen de las mismas 60 personas, así que no son independientes. La validación debe separar por usuario.
- **Exposición desigual.** No todos recorrieron los 28 nodos; por ejemplo, el nodo 24 solo lo vieron 20 sesiones. Que un nodo no aparezca no significa un valor cero.
- **Atípicos.** Las duraciones por nodo van de 0,42 s a 211,38 s. Se conservan y se reportan junto con la mediana y el promedio.
- **Muestra.** El respaldo trae 5 sesiones adicionales que se excluyen; se trabaja con los IDs de las 60 depuradas.

## Metodología

El ETL se hace en R. Primero se extraen y validan las tablas. Luego se resumen los eventos antes de unirlos, para no duplicar decisiones ni tiempos. Por último se unen proceso y decisiones por `session_id + node_id`, agregando `option_id` cuando el análisis es por alternativa.

El resultado son dos tablas de análisis, una para cada línea de trabajo:

| Línea | Tabla | Método | Evaluación |
|---|---|---|---|
| Perfiles del cursor | Por usuario | Clustering jerárquico (Ward) sobre indicadores estandarizados y ajustados por nodo: tiempo típico, cambios entre opciones y amplitud de exploración. Se comparan entre 2 y 4 grupos. | Silhouette, tamaño de los grupos y estabilidad ante remuestreo y distintos conjuntos de nodos |
| Predicción de la alternativa | Por decisión | Regresión logística multinomial y árbol de clasificación, usando el nodo y los eventos previos al clic | Matriz de confusión, F1 macro y exactitud, comparadas con la alternativa más frecuente (global y por nodo) |

## Análisis

| Notebook | Pregunta |
|---|---|
| [notebooks/01_telemetria_vs_mlq.ipynb](notebooks/01_telemetria_vs_mlq.ipynb) | ¿Las decisiones y la telemetría del cursor correlacionan con el MLQ-5X externo o permiten predecirlo? |

Requisitos: Python 3.10 o superior con `pandas`, `numpy`, `scipy`, `scikit-learn`, `statsmodels`, `matplotlib` y `seaborn`. Ejecutar desde la carpeta `notebooks/`; la corrida completa tarda unos 6 minutos.

## Plan de trabajo

| Fechas | Etapa |
|---|---|
| 9–22 oct. | Datos y pipeline: base depurada y variables del cursor |
| 23 oct.–5 nov. | Modelos y evaluación, con validación por usuario |
| 6–19 nov. | Dashboard BI: integración de resultados al informe individual |
| 20–26 nov. | Revisión, análisis de sensibilidad e informe final |
| 27 nov.–2 dic. | Preparación de la demostración |
| 3 dic. | Presentación final |

## Organización del trabajo

Las tareas se gestionan con issues de GitHub, usando las plantillas de [.github/ISSUE_TEMPLATE/](.github/ISSUE_TEMPLATE/) ("Tarea" y "User Story").
