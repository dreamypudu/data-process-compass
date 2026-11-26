# Guía simple de los notebooks

Esta guía explica, sin lenguaje técnico, qué hace cada notebook del proyecto, cómo funciona el modelo que usa y qué resultado dio. No hace falta saber programar ni estadística.

Los notebooks están en la carpeta `notebooks/`. Se guardan sin resultados: para ver las tablas y los gráficos hay que ejecutarlos (ver el final de esta guía).

## 1. El proyecto en un minuto

**COMPASS** es una simulación tipo juego. La persona hace de directora de un centro de salud (un CESFAM) y, en 28 situaciones distintas, debe elegir entre **tres alternativas: A, B o C**. En el piloto jugaron 60 estudiantes y egresados de carreras de la salud.

Mientras la persona decide, el juego registra dos cosas:

- **Qué eligió** en cada situación.
- **Cómo movió el mouse** antes de elegir: sobre qué alternativa lo puso, cuánto tiempo y cuántas veces cambió de una a otra.

Además existe un **cuestionario de liderazgo** (el MLQ), en el que cada persona se describe a sí misma. La idea del proyecto es que lo que alguien *hace* puede decir algo distinto de lo que alguien *dice de sí mismo*.

Cada notebook responde una pregunta:

| Notebook | Pregunta en simple |
|---|---|
| **01** | ¿Lo que la persona hace en el juego se parece a lo que dice de sí misma en el cuestionario? |
| **02** | ¿Hay "tipos" de personas según cómo mueven el mouse? |
| **03** | ¿Mirando el mouse unos segundos se puede adivinar qué alternativa va a elegir? |

## 2. Palabras que se repiten

| Palabra | Qué significa aquí |
|---|---|
| **Nodo** | Una de las 28 situaciones del juego. |
| **Alternativa** | Una de las tres opciones (A, B o C) de un nodo. |
| **Telemetría / datos de proceso** | Lo que registra el juego mientras la persona decide: movimientos del mouse y tiempos. |
| **Hover** | Cuando el mouse está encima de una alternativa. |
| **MLQ** | Cuestionario de liderazgo. Da puntajes en 9 dimensiones (por ejemplo, motivación inspiradora o consideración individualizada). Se asume que el archivo `puntajes_ext.csv` es ese cuestionario; el repositorio no lo documenta, así que conviene confirmarlo. |
| **Modelo** | Una fórmula que aprende patrones de los datos y los usa para estimar algo. |
| **Variable objetivo** | Lo que el modelo intenta adivinar. |

## 3. Cómo se prepararon los datos (igual en los tres notebooks)

Los archivos de `data/` ya venían depurados: se habían resuelto decisiones repetidas y sesiones de prueba (ver `data/README.md`). Encima de eso se hizo lo siguiente:

1. **Una fila por situación.** El archivo de movimientos del mouse repite la misma información en muchas filas; se dejó una por persona y nodo.
2. **Comprobación de coherencia.** La alternativa registrada en los movimientos coincide con la decisión registrada, en el 100 % de los casos.
3. **Comparar a cada persona con las demás en el mismo nodo.** Algunos nodos son más largos o difíciles que otros. Por eso cada medida del mouse (por ejemplo, el tiempo) se expresa como "más o menos que el resto de la gente *en ese mismo nodo*" y luego se promedia por persona.
4. **Casos ausentes.** No todos recorrieron los 28 nodos (el nodo 24 lo vieron solo 20 personas). Lo que falta se trata como ausente, nunca como cero.
5. **Casos extremos.** No se eliminaron; se usó escala logarítmica en los tiempos para que unos pocos valores muy altos no dominen.
6. **Variables descartadas.** "Eligió la última alternativa que miró" valía 1 en todos los casos (para hacer clic hay que pasar por encima de la opción), y el tiempo hasta el primer hover no informaba nada. Ninguna de las dos sirve.

## 4. Cómo evitamos engañarnos

Con solo 60 personas es muy fácil encontrar patrones que son pura casualidad. Por eso los tres notebooks usan las mismas reglas:

1. **Examen con preguntas nuevas.** El modelo nunca se evalúa con las mismas personas con las que aprendió. Se reparte a las personas en grupos, se entrena con unos y se prueba con otro, y se repite rotando. Se separa **por persona**, no por decisión, porque las decisiones de una misma persona se parecen entre sí.
2. **Compararse con una respuesta "tonta".** Un modelo solo vale si le gana a una regla muy simple, como "decir siempre el promedio" o "decir siempre la alternativa más común".
3. **Preguntarse si fue azar.** Se *revuelven las respuestas al azar* y se repite todo. Si con respuestas revueltas el modelo acierta casi igual que con las verdaderas, el resultado no significa nada. Es lo que se llama **prueba de permutación**.
4. **Corregir por hacer muchas pruebas.** Si se prueban 100 relaciones, alguna saldrá "significativa" por azar. Se aplica una corrección para tenerlo en cuenta.
5. **Hablar de lo exploratorio como exploratorio.** Con 60 personas, los resultados son pistas, no certezas.

---

## 5. Notebook 01: ¿lo que hacen se parece a lo que dicen?

**Archivo:** `notebooks/01_telemetria_vs_mlq.ipynb`

### Qué se quería saber
Si las decisiones y los movimientos del mouse en el juego permiten acercarse a los puntajes que cada persona obtuvo en el cuestionario de liderazgo.

### Qué se hizo, paso a paso
1. **Qué se intenta adivinar.** Los puntajes del cuestionario: 9 dimensiones. Como se parecen mucho entre sí, además se juntaron en 3 puntajes: *transformacional*, *transaccional* y *pasivo*.
2. **Dos fuentes de información sobre cada persona:**
   - **Decisiones:** cada alternativa del juego había sido etiquetada por expertos con su "tipo de liderazgo". Para cada persona se promedian las etiquetas de las alternativas que eligió.
   - **Mouse:** ocho medidas, como el tiempo por situación, cuántas veces cambia de alternativa, cuántas alternativas explora y si elige la primera que mira.
3. **Correlaciones.** Se miró si, cuando una medida sube, el puntaje del cuestionario tiende a subir o bajar (con un método que no exige que la relación sea una línea recta, llamado Spearman).
4. **Modelos.** Se probaron cuatro, de más simple a más complejo:
   - **El promedio** (la respuesta "tonta"): le da a todos el mismo puntaje.
   - **Ridge y Lasso:** son una línea de tendencia con "freno". Penalizan las fórmulas demasiado complicadas, para que el modelo no memorice las particularidades de 60 personas. Lasso, además, descarta por completo las medidas que no sirven.
   - **Random Forest:** muchos "árboles de preguntas de sí o no" que votan entre sí.
5. **Examen.** Las 60 personas se dividieron en 5 grupos: se entrenó con 4 y se probó con el quinto, rotando, y todo se repitió 10 veces con particiones distintas.
6. **Prueba de azar.** Se revolvió el cuestionario 300 veces para ver qué resultados da la pura casualidad.
7. **Qué usa el modelo.** Se midió cuánto empeora al desordenar cada variable.

### Qué se encontró
- **Las decisiones no se parecen al cuestionario.** Ninguna dimensión de la simulación correlaciona con la misma dimensión del cuestionario (todas las correlaciones son débiles, de −0,16 a +0,15).
- **Los tres puntajes agrupados no se pueden predecir.** Ningún modelo le gana al promedio.
- **Hay una pista en una dimensión: *consideración individualizada*.** Con solo los datos del mouse, el modelo logra una mejora pequeña: se equivoca 1,29 puntos en promedio, frente a 1,38 del promedio (sobre una escala de 16). Las personas con más puntaje tienden a **no elegir la primera alternativa que miran**, a elegir la que más revisaron, a cambiar menos de alternativa y a tomarse más tiempo. En otras palabras: **deliberan más**.

### Cómo leerlo
Es una asociación débil y exploratoria. La dimensión se eligió entre 12 posibles, y con 60 personas un efecto pequeño es difícil de confirmar. Sirve como hipótesis para probar con más participantes, no como herramienta para estimar el puntaje de una persona.

---

## 6. Notebook 02: ¿hay "tipos" de personas según el mouse?

**Archivo:** `notebooks/02_clustering_perfiles.ipynb`

### Qué se quería saber
Si las personas se dividen de forma natural en grupos según cómo exploran las alternativas.

### Qué tipo de modelo es
Aquí **no hay nada que adivinar**. El modelo recibe los datos y propone una agrupación por su cuenta. Se llama **clustering**. Usamos el método de **Ward**: empieza con cada persona sola y va juntando las más parecidas hasta quedar con 2, 3 o 4 grupos.

### Qué se hizo, paso a paso
1. **Dos formas de agrupar:**
   - **Por cómo deciden:** tiempo, cambios entre alternativas y cantidad de alternativas exploradas.
   - **Por qué deciden:** las alternativas A/B/C elegidas en los 24 nodos que todas las personas recorrieron.
2. **¿Los grupos son reales?** Un método de agrupación siempre entrega grupos, aunque no existan. Hay que preguntarse:
   - **¿Están bien separados?** Se mide con un número llamado *silhouette* (qué tan lejos están unos grupos de otros).
   - **¿Salen igual de separados con datos inventados?** Se simularon datos parecidos pero **sin grupos reales**. Si ahí también salen grupos igual de separados, los grupos observados no prueban nada.
   - **¿Se repiten con personas nuevas?** Aquí entra el "entrenar y probar": se forman los grupos con el 70 % de las personas y al 30 % restante se le asigna el grupo más parecido. Si los grupos son reales, esa asignación debería coincidir con los grupos que se forman usando a todos.
   - **Otras pruebas de estabilidad:** repetir con muestras al azar, usar solo la mitad de los nodos y probar con otro algoritmo (KMeans).
3. **Comparar los grupos con otras cosas:** el cuestionario MLQ, las decisiones del juego, y el día en que cada persona jugó (por si algo cambió entre semanas).
4. **Alternativa a los grupos:** un **índice continuo de exploración** (de menos a más) y el percentil de cada persona.

### Qué se encontró
- **Por qué deciden (las alternativas elegidas): no hay grupos.** Los resultados no se distinguen de datos mezclados al azar.
- **Por cómo deciden (el mouse): no hay "tipos" claros, sino una cola.** Con 2 grupos queda uno grande (51 personas) y uno pequeño (9). Las 9 exploran y cambian mucho menos: unos 2 cambios de alternativa por nodo, frente a 4,5 en el resto. Con 3 o 4 grupos no hay estructura.
- **Ese grupo pequeño es la cola de una distribución, no un tipo bien separado.** Supera a los datos inventados con una distribución simétrica, pero no cuando los datos inventados conservan la asimetría de los datos reales. Es un resultado en el límite.
- **No hay relación con el cuestionario.** Con 9 personas es difícil detectar diferencias, así que esto es "no hay evidencia", no "no hay relación".
- **Sí hay relación con las decisiones del juego.** Las 9 personas eligieron alternativas menos transformacionales y más cercanas a un liderazgo pasivo. La diferencia se mantiene al comparar solo a personas del mismo día, aunque el grupo pequeño no aparece en los últimos tres días de aplicación (conviene preguntar a quien aplicó el piloto si hubo algún cambio).

### Cómo leerlo
Para el dashboard conviene mostrar **un índice y un percentil** ("explora más que el 80 % de los participantes"), y no etiquetas de tipo. Los resultados quedan en `notebooks/output/perfiles_cursor.csv`.

---

## 7. Notebook 03: ¿se puede adivinar qué alternativa va a elegir?

**Archivo:** `notebooks/03_prediccion_alternativas.ipynb`

### Qué se quería saber
Si mirando el mouse durante los primeros segundos de una situación se puede anticipar qué alternativa (A, B o C) va a elegir la persona.

### La trampa principal
En **todas** las decisiones, la última alternativa sobre la que pasa el mouse es la que se elige. Si el modelo ve el movimiento completo, "adivina" el 100 %, pero solo porque ya vio la respuesta. Para que la prueba sea honesta se hizo lo siguiente.

### Qué se hizo, paso a paso
1. **Una "fotografía" del mouse hasta un instante `t`.** Se probaron cinco cortes distintos: 2, 4, 6, 8 y 12 segundos desde que aparece la situación. El modelo solo ve lo que pasó hasta ese instante.
2. **Solo decisiones que aún no se han tomado.** En el segundo `t` se predice únicamente para quienes todavía no eligieron, que es lo que se sabría en una aplicación real.
3. **Qué información usa el modelo:**
   - **El nodo:** de qué situación se trata.
   - **El mouse hasta `t`:** tiempo sobre cada alternativa, veces que entró a cada una, cambios, sobre cuál está el cursor en ese momento y cuál ha mirado más.
   - **La historia previa de la persona:** qué eligió antes (solo decisiones anteriores a la actual).
4. **Tres modelos:**
   - **Regresión logística:** calcula la probabilidad de A, B y C sumando "puntos a favor" de cada alternativa según lo que se observó.
   - **Árbol de decisión:** una cadena de preguntas de sí o no que termina en una predicción.
   - **Random Forest:** muchos árboles que votan.
5. **Las respuestas "tontas" con las que se compara:**
   - **Elegir siempre la más frecuente en general** (la A).
   - **Elegir siempre la más frecuente en ese nodo.** Es la comparación importante: es lo que se sabe solo con conocer la situación, sin mirar el mouse.
6. **Examen por personas,** igual que en los otros notebooks, con intervalos de confianza que remuestrean personas y no decisiones.
7. **Prueba de azar:** se mezclaron las alternativas elegidas dentro de cada nodo y se repitió el procedimiento.

### Qué se encontró
- **Conocer el nodo ya explica mucho.** La alternativa más común en cada nodo acierta entre 60 % y 62 %, frente a cerca de 50 % de elegir siempre A.
- **Los primeros segundos casi no ayudan.** Con cortes de 2 a 8 segundos, mirar el mouse no mejora de forma consistente. A los 2 segundos, el 88 % de las personas aún no ha puesto el cursor sobre ninguna alternativa.
- **A los 12 segundos aparece una mejora pequeña:** 63,7 % frente a 60,2 %. Pero 12 segundos ya es mitad de la decisión (la mediana de un nodo es de 18 s) y el resultado se eligió entre cinco cortes, así que no es firme.
- **El árbol de decisión funciona peor que la regresión logística** en todos los cortes; con tan pocas personas memoriza en vez de aprender.
- **La historia previa de la persona no aporta.**

### Cómo leerlo
La respuesta a la pregunta de la propuesta es: **la exploración temprana del cursor no permite anticipar la elección más allá de conocer la situación**. Observando hasta 12 segundos hay una mejora de unos 3,5 puntos, pero es pequeña.

---

## 8. Qué muestran los tres notebooks juntos

- Lo que la gente **dice** (cuestionario) y lo que **hace** (decisiones en el juego) se parecen poco.
- **Cómo** deciden (movimiento del mouse) aporta algo distinto de **qué** deciden: un indicio con la consideración individualizada (notebook 01) y una relación con la cola de personas que exploran poco (notebook 02).
- Ese "cómo" es más un **continuo** (de explorar menos a más) que una división en tipos, y no alcanza para anticipar la elección (notebook 03).

## 9. Límites

- **Solo 60 personas.** Los efectos pequeños son muy difíciles de confirmar.
- **Todo es exploratorio.** Se probaron muchas combinaciones; algunas decisiones (como la ventana de 12 s) se tomaron después de ver los resultados.
- **El cuestionario no está documentado en el repositorio.** Se asumió que `puntajes_ext.csv` es el MLQ.
- **Posible efecto de fecha.** El grupo de poca exploración se concentra en los primeros días del piloto.
- **Asociación no es causalidad.** Que el mouse se detenga sobre una alternativa no la causa.

## 10. Cómo ejecutarlos

Se necesita Python 3.10 o superior con `pandas`, `numpy`, `scipy`, `scikit-learn`, `statsmodels`, `matplotlib` y `seaborn`. Abrir cada notebook desde la carpeta `notebooks/` y ejecutar todas las celdas.

| Notebook | Tiempo aproximado | Archivo que guarda |
|---|---|---|
| 01 | 6 minutos | (ninguno) |
| 02 | 1,5 minutos | `notebooks/output/perfiles_cursor.csv` |
| 03 | 15 minutos | `notebooks/output/resultados_prediccion_alternativas.csv` |
