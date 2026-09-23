# Proyecto 1: ¿Compra el dinero los goles?

Economía, población y el desempeño de las selecciones nacionales de fútbol (1960–2025).

**Autor:** Milton Beltrán
**Curso:** Ciencia de Datos · Sección A · Segundo semestre 2026

EDA y modelo baseline sobre 49,547 partidos internacionales cruzados con los indicadores
económicos del Banco Mundial, para responder si el PIB per cápita explica el desempeño
futbolístico de un país y dónde cae Guatemala respecto a esa expectativa.

## Contenido

```
P1-Beltran.ipynb       el proyecto completo, ya ejecutado y con sus salidas
requirements.txt       dependencias del entorno del curso
data/
  procedencia.json     ficha de fuentes: URL, licencia, fecha de descarga y notas
  raw/                 los 11 archivos crudos, congelados al 2026-09-14
```

## Cómo correrlo

Requiere **Python 3.14** (este proyecto se ejecutó en 3.14.6).

```bash
python3 -m venv .venv
source .venv/bin/activate          # en Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

Después, abrir `P1-Beltran.ipynb` en VS Code o Jupyter, seleccionar ese entorno como kernel
y correr **Restart Kernel and Run All Cells**. También se puede desde la terminal:

```bash
jupyter nbconvert --execute --to notebook --inplace P1-Beltran.ipynb
```

Son 24 celdas de código y tarda alrededor de **2 segundos** en total.

## Notas para quien lo ejecute

- **Las rutas son relativas.** El notebook lee `data/raw/`, así que hay que correrlo con
  `P1-Beltran.ipynb` y la carpeta `data/` en el mismo directorio. No hace falta configurar nada más.
- **No se descarga nada al correr.** Los archivos de `data/raw/` son copias congeladas, así que
  el notebook funciona sin conexión y da siempre los mismos números. Las APIs del Banco Mundial
  y del ONS siguen vivas y sus valores pueden cambiar por revisiones estadísticas; por eso se
  versionaron las copias en lugar de bajarlas en tiempo de ejecución.
- **El resultado es determinista.** `RANDOM_STATE = 42` fija la partición train/test. Los números
  citados en el texto del notebook son los que salen al ejecutarlo: R² de test 0.1472, MAE de
  test 0.1489 contra 0.1621 del baseline trivial.
- De `requirements.txt` este proyecto solo usa `numpy`, `pandas`, `pyarrow`, `matplotlib`,
  `seaborn`, `scikit-learn` e `ipykernel`. El resto son dependencias de otras sesiones del curso
  y se incluyen para reproducir el entorno tal como está declarado en `data/procedencia.json`.

## Fuentes y licencias

| Fuente | Datos | Licencia |
|---|---|---|
| [martj42/international_results](https://github.com/martj42/international_results) | 49,547 partidos internacionales, 1872–2026 | CC0-1.0 (dominio público) |
| [Banco Mundial](https://data.worldbank.org) | PIB per cápita (US$ constantes de 2015) y población, 1960–2025 | CC BY 4.0 |
| [ONS, Reino Unido](https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates) | Población de Inglaterra, Escocia, Gales e Irlanda del Norte, 1971–2024 | Open Government Licence v3.0 |

La ficha completa, con URLs de descarga, diccionario de variables y las fuentes que se
evaluaron y se descartaron, está en `data/procedencia.json`.
