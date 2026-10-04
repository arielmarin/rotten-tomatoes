# Predicción de Recepción Crítica en Rotten Tomatoes

Proyecto final de Machine Learning — Ariel German Marin

## Descripción

Este proyecto construye un modelo de clasificación multiclase que
predice si una película será calificada como **Rotten**, **Fresh** o
**Certified-Fresh** en Rotten Tomatoes, utilizando exclusivamente
información disponible **antes del estreno** (género, clasificación
por edad, duración, elenco, dirección, guion, productora y fecha de
estreno).

El análisis está dirigido a estudios cinematográficos, distribuidoras
y equipos de marketing que necesitan anticipar la recepción crítica
esperada de un título, para tomar decisiones informadas sobre
presupuesto de promoción, cantidad de salas de estreno o fecha de
lanzamiento.

## Dataset
Fuente: ["Rotten Tomatoes Movies and Critic Reviews"](https://www.kaggle.com/datasets/stefanoleone992/rotten-tomatoes-movies-and-critic-reviews-dataset)

El dataset original no está incluido en este repositorio por su
tamaño. Para reproducir el proyecto:

1. Descargar `movies.csv` (y opcionalmente `critics.csv`) desde el
   link de Kaggle de arriba.
2. Colocarlos en `data/raw/`.

## Estructura del repositorio

rotten-tomatoes-ml-project/
├── data/
│ ├── raw/ # CSVs originales (no incluidos, ver arriba)
│ └── processed/ # Datos limpios generados por el notebook
├── notebooks/
│ └── proyecto_final.ipynb
├── requirements.txt
└── README.md

## Cómo reproducir el proyecto

```bash
git clone https://github.com/<tu-usuario>/rotten-tomatoes-ml-project.git
cd rotten-tomatoes-ml-project

python -m venv venv
venv\Scripts\activate          # Windows
# source venv/bin/activate     # Mac/Linux

pip install -r requirements.txt
```

Descargar los datos como se indica arriba, colocarlos en
`data/raw/`, y correr `notebooks/proyecto_final.ipynb` de principio a
fin (Restart & Run All).

## Metodología

1. **Lectura de datos y EDA**: carga del dataset, análisis univariado,
   bivariado y multivariado de las variables frente al target.
2. **Data Wrangling**: tratamiento de duplicados, nulos (estrategia
   justificada por columna) y outliers.
3. **Feature Engineering**: multi-hot encoding de géneros, extracción
   de componentes de fecha, agrupación de productoras y
   clasificaciones poco frecuentes, métricas resumen de elenco/equipo.
4. **Prevención de leakage**: se excluyeron explícitamente todas las
   columnas que representan información posterior al estreno
   (`tomatometer_rating`, `audience_rating`, conteos de reseñas, etc.).
5. **Modelado**: comparación de Regresión Logística, Árbol de
   Decisión, Random Forest y XGBoost, dentro de un `Pipeline` con
   preprocesamiento (`OneHotEncoder`) para evitar fugas de
   información entre train y test.
6. **Optimización**: ajuste de hiperparámetros con `GridSearchCV` y
   validación cruzada estratificada (5 folds).
7. **Explicabilidad**: interpretación del modelo final con SHAP.

## Resultados

| Modelo | F1 macro (test) |
|---|---|
| Baseline | 0.202 |
| Regresión Logística | 0.476 |
| Árbol de Decisión | 0.521 |
| Random Forest | 0.546 |
| Random Forest optimizado | 0.541 |
| XGBoost | 0.552 |
| **XGBoost optimizado (modelo final)** | **0.561** |

**Modelo final:** XGBoost con hiperparámetros optimizados.
**Métrica principal:** F1 macro (por el desbalance moderado entre
clases: 43% Rotten, 37% Fresh, 19% Certified-Fresh).

### Principales hallazgos

- `release_year` y `runtime` son las variables de mayor peso según
  SHAP, capturando relaciones no lineales que la correlación lineal
  no detecta.
- El género es un fuerte predictor: Documentary y Art House &
  International se asocian a mejor recepción; Horror y Comedy, a peor.
- La clasificación por edad NR se asocia a mejor recepción; PG-13 es
  consistentemente la más castigada por la crítica.
- El modelo distingue bien entre recepción "buena" y "mala", pero
  tiene más dificultad para separar Fresh de Certified-Fresh entre sí.

## Limitaciones

- Dataset con corte a octubre de 2020, no incluye estrenos posteriores.
- Probable sesgo de supervivencia en películas antiguas (solo los
  clásicos reconocidos permanecen catalogados).
- Al excluir variables posteriores al estreno (para evitar leakage),
  se descartan señales potencialmente útiles (presupuesto, marketing).

El detalle completo del análisis, código y conclusiones está en
[`notebooks/proyecto_final.ipynb`](notebooks/proyecto_final.ipynb).