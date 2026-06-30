# Predicción de visibilidad atmosférica a partir de PM10 — Piura, Perú
Modelado predictivo de la visibilidad atmosférica (km) en función de la concentración de material particulado PM10, usando datos de tres estaciones de monitoreo en Piura, Perú (26 de Octubre, UDEP y Piura). El proyecto compara cinco algoritmos de regresión y selecciona el de mejor desempeño mediante validación cruzada y ajuste de hiperparámetros.

## Objetivo
Evaluar en qué medida la concentración de PM10 permite predecir la visibilidad atmosférica, comparando el desempeño de distintos modelos de regresión bajo un mismo pipeline de preprocesamiento.

## Datos
- Fuente: mediciones horarias/diarias de PM10 (µg/m³) y rango visual / visibilidad (km) en tres estaciones de monitoreo de Piura.
- Estaciones: 26 de Octubre, UDEP, Piura.
- Cada estación cuenta con dos sensores (A y B); tras un análisis comparativo de varianza y estabilidad, se seleccionó un único sensor por variable como fuente oficial (PM10: sensor A, Visibilidad: sensor B), descartando el resto.
- Tras la limpieza y combinación de ambas variables, el dataset final cuenta con 776 registros utilizables. La estación Piura presenta un alto porcentaje de datos faltantes (~86%), por lo que el modelo aprende mayoritariamente de las estaciones 26 de Octubre y UDEP.

## Metodología
1. Carga y exploración de datos: comparación de sensores A/B por estación (series temporales, estadísticos descriptivos) y selección del sensor oficial.
2. Preprocesamiento:
   - Transformación a formato largo (long format) por estación.
   - Manejo de valores nulos mediante interpolación temporal (`method="time"`) con relleno de extremos (`ffill`/`bfill`).
   - Detección y tratamiento de outliers en PM10 con el método IQR, calculado **solo sobre el set de entrenamiento** para evitar fuga de información (data leakage).
   - División train/test temporal por estación (80/20), respetando el orden cronológico.
   - Escalado de variables numéricas (`StandardScaler`) y codificación one-hot de la estación (`OneHotEncoder`), integrados en un `ColumnTransformer`.
3. Modelado: cinco modelos de regresión, cada uno dentro de un `Pipeline` (preprocesador + estimador), con optimización de hiperparámetros vía `GridSearchCV` y validación cruzada de 5 folds (`KFold`, `shuffle=True`):
   - Regresión Lineal Múltiple
   - K-Nearest Neighbors (KNN) Regressor
   - Random Forest Regressor
   - Gradient Boosting Regressor
   - Support Vector Regression (SVR)
4. Evaluación:
   - Métricas en el set de prueba: RMSE, MAE y R².
   - Validación cruzada sobre el set de entrenamiento para detectar sobreajuste (comparando RMSE de test vs. RMSE promedio de CV).
   - Visualización de valores reales vs. predichos por modelo, y gráficos comparativos de RMSE/R².

## Resultados
Todos los modelos evaluados alcanzaron un R² en el set de prueba entre 0.867 y 0.882, lo que indica que el PM10 por sí solo explica la gran mayoría de la variabilidad de la visibilidad atmosférica, una relación consistente con el fenómeno físico (a mayor concentración de partículas, menor visibilidad).

El notebook genera una tabla comparativa final (RMSE, MAE, R²) ordenada de mejor a peor desempeño, junto con gráficos de dispersión real vs. predicho y barras comparativas por modelo. Consulta la sección "4. Evaluación" y "Conclusiones" del notebook para el detalle completo.

## Estructura del proyecto
```
.
├── Modelado_PM10_Visibilidad_Piura.ipynb   # Notebook principal (análisis, modelado, evaluación)
├── raw-pm10-gm.csv                          # Datos de PM10
├── visual-range-km.csv                      # Datos de visibilidad
└── README.md
```

## Requisitos
- Python 3.9+
- pandas
- numpy
- matplotlib
- seaborn
- scikit-learn

Instalación rápida:
```bash
pip install pandas numpy matplotlib seaborn scikit-learn
```

## Cómo ejecutarlo
1. Clona el repositorio:
   ```bash
   git clone [https://github.com/JoaoSanchez14/Prediccion-visibilidad-pm10-peru.git](https://github.com/JoaoSanchez14/Prediccion-visibilidad-pm10-peru.git)
   cd Prediccion-visibilidad-pm10-peru
   ```
2. Coloca `raw-pm10-gm.csv` y `visual-range-km.csv` en la ruta esperada por el notebook (o ajusta las rutas en la celda de carga de datos).
3. Abre el notebook con Jupyter:
   ```bash
   jupyter notebook Modelado_PM10_Visibilidad_Piura.ipynb
   ```
4. Ejecuta las celdas en orden, de principio a fin.

## Conclusiones principales
- La estación Piura aporta poca información real debido a su alto porcentaje de datos faltantes; las conclusiones se sustentan principalmente en las estaciones 26 de Octubre y UDEP.
- Existe una relación fuerte y consistente entre PM10 y visibilidad atmosférica, capturada de forma similar por todos los modelos evaluados (R² entre 0.867 y 0.882).
- El uso de un pipeline único de preprocesamiento, validación cruzada y `GridSearchCV` permite comparar los modelos de forma justa y reduce el riesgo de sobreajuste en la selección de hiperparámetros.

## Autor
Proyecto desarrollado por Eduardo Joao Sánchez Farías como parte de un análisis de calidad del aire y visibilidad atmosférica en Piura, Perú.
