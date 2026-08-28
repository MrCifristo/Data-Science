# Ciencia de Datos · Sección A · Segundo semestre 2026

Notebooks del curso: apuntes de las sesiones de clase y hojas de trabajo resueltas.

**Milton Beltran**

---

## Estructura

```
.
├── 02. NumPy (23 de julio)/          # sesión 2: arrays, vectorización, broadcasting
├── 03. Pandas y SQL (28 de julio)/   # sesión 3: pandas, duckdb, polars, parquet
├── hdt1/                             # HDT 1: Pandas, SQL y DuckDB
├── HDT2/                             # HDT 2: Probabilidad, MLE y MAP
├── HDT3/                             # HDT 3: Inferencia, Naive Bayes y Regresión
├── requirements.txt                  # entorno único para todo el repo
└── README.md
```

Cada carpeta de hoja de trabajo tiene el enunciado en PDF y el notebook con la resolución.

## Entorno

Un solo entorno virtual sirve para todos los notebooks. Probado con **Python 3.14.7**.

```bash
python3 -m venv .venv
source .venv/bin/activate          # zsh/bash;  en Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

El `.venv/` está en el `.gitignore`, así que no se sube al repositorio: cada máquina lo reconstruye
con el comando de arriba.

### Correr los notebooks

**VS Code:** abrir el notebook, hacer clic en *Select Kernel* (esquina superior derecha) y elegir el
intérprete de `.venv`. VS Code lo detecta solo por estar en la raíz del repositorio.

**Jupyter:** registrar el kernel una sola vez y luego seleccionarlo desde la interfaz.

```bash
source .venv/bin/activate
python -m ipykernel install --user --name ciencia-datos --display-name "Python (ciencia-datos)"
```

Antes de entregar cualquier hoja: **Kernel → Restart & Run All**. Un notebook que no corre completo
de arriba a abajo pierde 0.5 pts.

### Dependencias

| Paquete | Para qué |
|---|---|
| `numpy`, `pandas`, `pyarrow` | base numérica y de datos (todas las sesiones y hojas) |
| `duckdb`, `polars`, `lxml` | SQL sobre DataFrames y formatos columnares (sesión 3, HDT1) |
| `requests` | descarga de datos desde APIs (HDT2) |
| `matplotlib`, `seaborn` | gráficas y datasets de ejemplo (HDT2, HDT3) |
| `scikit-learn` | regresión lineal (HDT3) |
| `ipykernel` | kernel de Jupyter para VS Code |

Las versiones están fijadas en `requirements.txt`. El archivo `hdt1/requirements.txt` es el original
que venía con esa hoja; el de la raíz lo reemplaza y cubre todo el repositorio.

> **Nota:** varios notebooks descargan datos en tiempo de ejecución (`penguins` y `tips` desde
> seaborn-data, `diamonds` desde seaborn, y el clima desde la API de Open-Meteo), así que hace falta
> conexión a internet para correrlos.

## Hojas de trabajo

| Hoja | Tema | Contenido |
|---|---|---|
| **HDT 1** | Pandas, SQL y DuckDB | Limpieza y máscaras sobre `penguins`, `groupby` + `agg`, columna derivada y `merge`; las mismas consultas en SQL con DuckDB (`GROUP BY` + `HAVING`, `JOIN`, `RANK() OVER`). Anexo de repaso de NumPy y `tips`. |
| **HDT 2** | Probabilidad, MLE y MAP | Descarga del clima de Ciudad de Guatemala (92 días, mayo–julio 2026) desde Open-Meteo con su ficha de procedencia; MLE de Bernoulli y de la Normal a mano, MAP con prior Beta(2,10) y cómo el prior se diluye conforme entran más datos. |
| **HDT 3** | Inferencia, Naive Bayes y Regresión | Test de permutación y Bonferroni sobre un A/B de 50,000 visitantes, un clasificador de spam contando con suavizado de Laplace, y OLS a mano sobre `diamonds` (incluye el arreglo log-log y regresión múltiple). |

### Datos generados

La HDT2 escribe en su propia carpeta al ejecutarse:

- `HDT2/clima_procedencia.json` — ficha de procedencia (fuente, URL, rango de fechas, filas).
- `HDT2/clima_gt.csv` — los datos descargados. Los `.csv` están en el `.gitignore`, así que este
  archivo se regenera corriendo el notebook.
