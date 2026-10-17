# Proyecto 2: diseño

**Autor:** Milton Beltrán · **Fecha:** 2026-10-16 · **Entrega:** hoy, 23:59
**Entregable:** `Proyecto 2/P2_BeltranMilton.ipynb`, sobre la plantilla `22. Lab Proyecto 2 (8 de octubre).ipynb`

---

## 1. Intención

El Proyecto 1 concluyó que el PIB per cápita explica poco del desempeño anual de una selección
(R² de test 0.147) y que la población pesa más que el dinero. El Proyecto 2 baja de país-año a
**partido** y hace una pregunta más dura:

> **Antes de que se juegue un partido internacional, ¿se puede predecir si gana el local, empatan o
> gana la visita? Y si ya sé la fuerza deportiva de cada equipo, ¿saber su economía mejora la
> predicción?**

Es la continuación del P1: si el dinero no agrega nada una vez que se conoce el nivel deportivo,
eso refuerza que la brecha entre países se explica en la cancha, no en el PIB.

**Por qué esta opción y no repetir la pregunta del P1:** se comparó con la opción de mantener el
target país-año (`winrate`) y añadir un ensamble. Se eligió la de partidos porque encaja mejor con la
rúbrica: clases desbalanceadas (la discusión de la métrica es real), 35 mil filas con interacciones
(el ensamble tiene dónde lucirse), fechas (validación temporal honesta) y una pregunta nueva que no
es el P1 con otro modelo.

## 2. Objetivos

- Cumplir las 9 secciones de la plantilla y los 10 puntos de la rúbrica.
- Validación sin fugas en todo el notebook: el test se toca una sola vez.
- Responder con un número si la economía agrega algo sobre el Elo.
- Que el clustering explique algo del modelo (dónde se equivoca más), que es lo que sube la nota.
- Notebook ejecutado con Restart & Run All, sin errores, contadores desde 1.

**No-objetivos**
- No repetir el EDA del P1. La sección 2 es corta y remite al P1.
- No perseguir el accuracy más alto: predecir fútbol es difícil, y un modelo que le gana al tonto
  por poco es un resultado válido si está bien medido.
- No cambiar de fuentes ni descargar nada nuevo.

**Anti-objetivos (sería un fracaso aunque corra)**
- Que el Elo o cualquier columna use información del propio partido o de partidos futuros.
- Elegir hiperparámetros o comparar modelos mirando el test.
- Leer importancias como causas ("subir el PIB hace ganar").
- Grupos sin nombre o clustering sin escalar.
- Que la sección 8 sean dos tareas pegadas sin conexión.

## 3. Restricciones

- Tiempo: entrega hoy a las 23:59. Objetivo interno: notebook completo hacia las 22:30.
- Entorno: el `.venv` de la raíz del repo (Python 3.14, scikit-learn 1.9.0, pandas 3.0.5). Sin
  librerías nuevas.
- Trabajo **parte por parte**: se construye una sección, Milton la revisa y se sigue. Nunca el
  notebook entero de un solo.
- AI con bitácora obligatoria (sin ella el resultado vale la mitad). La bitácora debe reflejar el
  reparto real: Claude escribe la mayor parte del código; Milton decide y redacta.

## 4. Datos

- Los mismos archivos congelados del P1 (descargados el 2026-09-14): `results.csv` (martj42,
  CC0-1.0), PIB per cápita `NY.GDP.PCAP.KD` y población `SP.POP.TOTL` (Banco Mundial, CC BY 4.0),
  población de las naciones británicas (ONS, OGL v3.0).
- Se copian a `Proyecto 2/data/raw/` para que la entrega funcione sola. La procedencia se cita en la
  sección 1 y se dice explícitamente qué se mejoró respecto al P1.
- Se reutiliza la lógica de limpieza del P1: filtro de agregados del Banco Mundial, diccionario de
  33 nombres, población ONS + PIB del Reino Unido para las cuatro naciones británicas, exclusión de
  selecciones no-FIFA.
- **Una fila = un partido** desde 1960 en el que ambos equipos tienen PIB y población ese año.
  Estimado: ~35,000 partidos (34,994 sin las británicas).
- **Target:** `resultado` ∈ {local, empate, visita} en tiempo reglamentario + extra (sin penales).
  Reparto medido: 48.6% local, 27.1% visita, 24.2% empate.

## 5. Columnas del modelo

Todas tienen que existir **antes** del partido.

| Columna | Qué es | Por qué |
|---|---|---|
| `elo_local`, `elo_visita` | Rating Elo de cada equipo justo antes del partido | La fuerza deportiva; reemplaza a `fuerza_rival` del P1 |
| `dif_elo` | `elo_local − elo_visita` | La señal principal esperada |
| `dif_log_pib` | log PIB per cápita local − visita | La pregunta del proyecto |
| `dif_log_pob` | log población local − visita | El hallazgo más fuerte del P1 |
| `neutral` | Si se juega en cancha neutral | Sin local real, la ventaja desaparece |
| `tipo_torneo` | Amistoso / eliminatoria / torneo continental / Mundial | No se juega igual un amistoso que un Mundial |

**El Elo** sigue la fórmula del World Football Elo Ratings: todos arrancan en 1500, ventaja de
local de +100 si no es cancha neutral, K por importancia (60 Mundial, 50 torneo continental, 40
eliminatorias y otros torneos mayores, 30 otros torneos, 20 amistosos) y multiplicador por diferencia
de goles. Se calcula recorriendo **todos** los partidos desde 1872 en orden cronológico (para que en
1960 los ratings ya estén calentados), y cada fila guarda el rating **anterior** a su partido.

**Mejora sobre el P1:** el Elo elimina las dos debilidades que el P1 declaró: la fuga de
`fuerza_rival` (usaba partidos futuros y de países de test) y que un promedio histórico plano no
ajustaba bien por la fuerza del rival.

## 6. Validación y modelos

- **Corte temporal:** train = 1960–2021, **test = partidos desde 2022** (~3,400). Sin barajar.
- **Validación cruzada dentro del train** con `TimeSeriesSplit` (filas ordenadas por fecha).
- Todo lo que se calcula con datos (escalado, one-hot, rellenos) va dentro de un `Pipeline` con
  `ColumnTransformer`, y el `Pipeline` dentro de la validación y del buscador.
- **Tonto:** `DummyClassifier(strategy="most_frequent")`, siempre "gana el local".
- **Simple:** regresión logística multinomial.
- **Ensamble:** gradient boosting (`HistGradientBoostingClassifier`) con `RandomizedSearchCV`;
  cada rango de hiperparámetros con su razón escrita.
- **¿Agrega algo el dinero?** Se compara en validación el mismo modelo con y sin `dif_log_pib` y
  `dif_log_pob`, tanto en la logística como en el boosting (con los hiperparámetros ganadores de la
  búsqueda, sin re-buscar para la versión sin economía). La diferencia se mide en CV, no en test.
- **Regla para elegir qué va a test, fijada antes de tocarlo:**
  - El modelo final es el de mayor F1 macro en CV entre la logística y el boosting.
  - Las columnas económicas se quedan en el modelo final solo si en CV mejoran el F1 macro por más
    de una desviación estándar entre folds; si no, el modelo final va sin ellas y eso es la respuesta.
  - En la sección 6 se reentrenan los tres (tonto, simple y el final) con todo el train y se evalúan
    en test **una sola vez**, para llenar la tabla de la plantilla. Las predicciones del modelo final
    son las que usan las secciones 6 y 8.

## Cancha neutral

En `results.csv`, en un partido en cancha neutral el "local" es solo el equipo listado primero. Se
conservan esas filas (son el 26.5%) y la etiqueta se lee como "equipo listado primero"; la columna
`neutral` le dice al modelo que ahí no hay ventaja real. En la sección 2 se reporta el reparto de
resultados por separado para cancha neutral y con local, para que el 48.6% no se malinterprete.

## 7. Métrica

- **Principal: F1 macro.** El tonto saca ~48.6% de accuracy sin predecir jamás un empate ni una
  victoria visitante. El error caro es ignorar dos de las tres clases; F1 macro pesa las tres por
  igual y deja al tonto donde corresponde.
- **Secundaria: log-loss**, que mide qué tan buenas son las probabilidades (lo que usaría quien
  quiera saber "¿qué chance tiene Guatemala?").
- **Dónde falla:** matriz de confusión, errores según la diferencia de Elo y el caso de los empates.

## 8. Parte sin respuestas: clustering de partidos

- Ruta A (agrupar) con k-Means sobre el **perfil del partido**: diferencia de Elo (en valor
  absoluto y con signo), diferencia económica, cancha neutral y tipo de torneo. Escalado obligatorio.
- El número de grupos se elige con silhouette **y** con que los grupos tengan sentido futbolístico.
- Cada grupo con nombre y descripción, por ejemplo "David contra Goliat", "parejo entre potencias",
  "amistoso entre chicos" (los nombres finales salen de los datos).
- El escalador y el k-Means se ajustan con train y se asignan al test.

## 9. Conexión (sección 8 del notebook)

Cruzar los grupos con los aciertos y errores del modelo en test (las predicciones de la sección 6,
sin volver a ajustar nada): F1 por grupo y en qué grupo se concentran los errores. Hipótesis a
verificar, no a suponer: el modelo falla más en los partidos parejos, que es donde viven los empates.

## 10. Conclusiones y límites

Tres párrafos según la plantilla: la respuesta con su número, lo que **no** se puede concluir
(correlación ≠ causa, importancias ≠ causas, sesgo hacia países con datos del Banco Mundial,
penales no cuentan) y qué haría falta para mejorarlo.

## 11. Reparto del trabajo

- **Claude:** código de cada sección; borrador de las **justificaciones técnicas** (por qué se
  preparó así, por qué esos rangos, por qué esa métrica), que Milton revisa y ajusta; cifras clave en
  el chat para la redacción; borrador de la bitácora basado en lo que de verdad pasó en la sesión.
- **Milton:** aprueba cada sección antes de seguir, toma las decisiones abiertas y escribe las
  **interpretaciones**: qué significan los resultados, los nombres y descripciones de los grupos y
  las conclusiones. En el notebook esos espacios quedan marcados con `✍️ (Milton)`.

## Decisiones de Claude (revisables)

- Copiar `data/raw` del P1 a `Proyecto 2/data/raw`.
- Test desde 2022 y `TimeSeriesSplit` en el train.
- F1 macro como principal y log-loss como secundaria.
- `HistGradientBoostingClassifier` como ensamble.
- Clustering de partidos, no de países.
- La regla de qué modelo va a test (sección 6) y el tratamiento de cancha neutral.
- Este documento vive en `Proyecto 2/diseño.md`.
