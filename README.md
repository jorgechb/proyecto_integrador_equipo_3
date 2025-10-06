# Proyecto: Servicio de calidad aumentrada por inteligencia artificial.

## Equipo 3:

- Jorge Chávez Badillo A01749448
- Andrea Fernanda Molina Blandón A00827133
- Ismael Alexander Carvajal González A01793925

## Tabla de avances:

| **Nombre del avance** | **Descripción**                                                            | **Link al recurso de avance**              |
| --------------------- | -------------------------------------------------------------------------- | ------------------------------------------ |
| Avance 0              | Breve descripción del avance 1.                                            | [Ver avance](https://link-al-avance-1.com) |
| Avance 1              | EDA y preprocesamiento de casos de prueba. Regression Test Selection (RTS) | [Ver avance](https://link-al-avance-2.com) |
| Avance 2              | Breve descripción del avance 3.                                            | [Ver avance](https://link-al-avance-3.com) |

## Contexto del ptoyecto y alcance

Tras revisión con IBM, el proyecto se centra en RTS (y opcionalmente un asistente de PR en fases futuras).
El EDA se reduce a lo esencial para alimentar el RTS y acelerar la ejecución.

## Estructura del repositorio

.
├─ notebooks/
│ └─ EDA*TestCases_MVP.py # Script ejecutable (sin Jupyter)
├─ data/
│ ├─ raw/
│ │ ├─ TestCases.csv # Casos de prueba (texto + metadatos)
│ │ ├─ ChangedFiles.csv # Archivos tocados por PR (ver esquema)
│ │ └─ PipelineMeta.csv # Metadatos del PR/CI (ver esquema)
│ └─ processed/
│ ├─ TestCases_clean_features.csv
│ ├─ RTS_selected_PR-\*.csv # Seleccionados por PR (salida)
│ ├─ RTS_selected_all*_.csv # Seleccionados de todos los PR (reporte final)
│ └─ RTS*report*_.xlsx # (opcional) Reporte Excel si hay openpyxl
└─ run_artifacts/
├─ RUN_OK.txt
└─ run_report.json # Evidencias y hashes de entrada

## Contenido principal

notebooks/EDA_TestCases_MVP.py
Pipeline completo: carga + normalización vectorizada, EDA mínimo, documento por PR (tokens de ruta, módulos, líneas cambiadas, severidad, migración DB), TF-IDF conjunto (tests+PR) y ranking RTS (coseno + señales de módulo, riesgo y tamaño).

## Esquemas de entrada (mínimos)

data/raw/TestCases.csv (columnas clave; se renombran por alias):
case_id, title, preconditions, steps, expected_result, obtained_result, observations, exec_count, test_status, feature_name, sprint, created_at, module

data/raw/ChangedFiles.csv:
pr_id, commit_id, file_path, file_type, lines_added, lines_deleted, component, module, risk_tag

data/raw/PipelineMeta.csv:
pr_id, pr_title, pr_desc, author, branch, commit_count, changed_files, ci_provider, service_affected, weekday, time_of_day, has_db_migration, severity_declared, test_history_fail_rate, build_id

Si faltan columnas extra, el script sigue funcionando con lo disponible (rellenos seguros).

## Flujo (resumen)

Carga y normaliza (vectorizado; sin apply por fila).

EDA esencial (solo lo útil para RTS):

Frecuencia de module_inferred (top 12).

Histogramas de longitudes (title_len, steps_len, expected_result_len) y exec_count.

## Documento por PR:

pr_text = pr_title + pr_desc + tokens(file_path) + módulos.

total_lines_changed = lines_added + lines_deleted por PR.

RTS score por PR:

rts*score = 0.6 * text*sim + 0.2 * module*match + 0.1 * risk + 0.1 \_ change_size

risk +0.2 si has_db_migration = yes.

change_size = min(1, total_lines_changed / 300).

Selección top-k con k = min(len, max(5, round(fraction\*len))).

## Salidas:

data/processed/RTS_selected_PR-<ID>.csv (por PR).

Reporte final: RTS*selected_all*_.csv y (si hay openpyxl) RTS*report*_.xlsx.

Evidencias: run_artifacts/run_report.json (hashes, shape, etc.).

Configuración (en el código)

Clase Config centralizada:

@dataclass
class Config:
RAW_TESTS = Path("../data/raw/TestCases.csv")
RAW_CHANGED = Path("../data/raw/ChangedFiles.csv")
RAW_META = Path("../data/raw/PipelineMeta.csv")
OUT_DIR = Path("../data/processed")
RUN_DIR = Path("../run_artifacts")
MAKE_PLOTS = True
USE_SVD = False # RTS no lo requiere (más rápido/ligero)
TOP_FRACTION = 0.15 # % de casos a seleccionar por PR
TFIDF_MAX_FEATURES = 6000
LINES_NORM = 300 # normalizador del tamaño del cambio
W_SIM, W_MODULE, W_RISK, W_SIZE = 0.6, 0.2, 0.1, 0.1

Para PR críticos o con migración de DB, puedes subir TOP_FRACTION a 0.25–0.30.

## Requisitos

Python ≥ 3.9

Instalar dependencias:

pip install -r requirements.txt

# (opcional para Excel)

pip install openpyxl

Ejecución
Como script (sin Jupyter)
python notebooks/EDA_TestCases_MVP.py

En notebook/Jupyter

Ejecuta todas las celdas y, al final, corre el reporte:

# Para todos los PR detectados

initial_cases, selected_cases, summary_by_pr, not_selected = build_rts_report()

# O para un PR concreto y con más cobertura (p.ej., crítico)

\_ = build_rts_report(pr_ids=["PR-1002"], top_frac=0.30)

Visualizaciones

EDA mínimo:

Frecuencias de module_inferred (top 12).

Histogramas de longitudes y exec_count.

## Diagnóstico RTS (por PR):

Histograma de rts_score con umbral.

Dispersión text_sim vs change_size (marcadores por module_match).

Barras de módulos dentro del top-k.

Para acelerar en CI, usa Config.MAKE_PLOTS = False.

Buenas prácticas aplicadas

Vectorización (sin apply por fila) en normalización de estados y heurística de módulos.

TF-IDF conjunto (tests+PR) y producto disperso matriz-vector para similitud (coseno).

Señales globales por PR (riesgo, tamaño) sin recomputar por fila.

Configuración centralizada y pesos explícitos (auditable).

EDA mínimo enfocado en RTS (menos tiempo y ruido).

Top-k acotado para datasets pequeños: k = min(k, len(full)).

## Solución de problemas

Log “top 5/4” → solucionado con acotación de k.

name 'rts_score_for_pr' is not defined → el reporte detecta y usa rts_rank_for_pr(...) automáticamente.

Excel no disponible → instala openpyxl o usa los CSV generados.

Selección vacía o poco creíble → amplía TestCases.csv con casos de los módulos tocados por los PR, ajusta TOP_FRACTION y mejora path_tokens con sinónimos de dominio (e.g., taxes→“impuestos”).

## Próximos pasos

Diccionario de dominio para tokenizar rutas/servicios (mejora text_sim).

Cobertura mínima por módulo impactado (“al menos 1 caso” por módulo tocado).

Feedback loop (futuro): resultados de ejecución → reajuste de pesos/umbral.

Integración con Watson Discovery (RAG) para enriquecer pr_text con contexto documental.

## Créditos

Equipo 3 — Proyecto Integrador (Castor, metalurgia para construcción).
RTS MVP implementado con enfoque QA/IA, priorizando eficiencia, consistencia y extensibilidad.
