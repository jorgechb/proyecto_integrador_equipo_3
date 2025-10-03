# Proyecto Integrador

## Equipo 3:

- Jorge Chávez Badillo A01749448  
- Andrea Fernanda Molina Blandón A00827133  
- Ismael Alexander Carvajal González A01793925

---

# Avance1.#Equipo — Análisis Exploratorio de Datos (EDA)

Este repositorio contiene el *Avance 1* del proyecto: **EDA + preprocesamiento** sobre el histórico de **casos de prueba** del proyecto Cotizaciones/Pedidos.

## Contenido

- `notebooks/EDA_TestCases_MVP.py`: script ejecutable (sin Jupyter) con el EDA y preprocesamiento.
- `data/raw/TestCases.csv`: dataset de ejemplo (extraído del HU1).
- `data/processed/TestCases_clean_features.csv`: dataset limpio + features (se genera al correr el script).
- `CONCLUSIONES.md`: plantilla para documentar hallazgos, 2.1 selección de features y 2.2 correcciones.

## Qué cubre (rúbrica)

- **Estructura**: shape, dtypes, faltantes, frecuencias de categorías.
- **Univariante**: histogramas y boxplots.
- **Bi/Multivariante**: correlaciones numéricas (heatmap), crosstab y **Cramér’s V** para categóricas.
- **Preprocesamiento (2.2)**: normalización textos/categorías, estandarización de estado, winsorización y `log1p` en duración, reducción de cardinalidad (Top-N + “otros”), deduplicación con justificación.
- **Selección de características (2.1)**: variables clave para recuperación/priorización + TF-IDF + **SVD** (LSA) → `svd_1..k`.
- **Conclusiones**: plantilla con preguntas guía.

## Requisitos

- Python >= 3.9  
- `pip install -r requirements.txt`

## Ejecución

```bash

python notebooks/EDA_TestCases_MVP.py
```

---

# Avance 2. Ingeniería de características



