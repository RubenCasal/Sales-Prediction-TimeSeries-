# Sales Prediction with Time Series Analysis

Este proyecto aborda el desafío propuesto por una competencia de Kaggle, donde se busca predecir las ventas de distintas familias de productos en múltiples tiendas de Favorita, una cadena ubicada en Ecuador. Utilizando técnicas de series de tiempo, el objetivo es desarrollar un modelo que permita estimar las ventas futuras a partir de datos históricos, lo cual es crucial para mejorar la planificación de inventarios, gestionar promociones y optimizar el suministro de productos.

La solución incluye una serie de modelos predictivos que se entrenan con datos que incorporan factores como promociones, transacciones históricas, fluctuaciones de precios de petróleo y eventos especiales. Este enfoque permite capturar los patrones y tendencias en las ventas, ofreciendo una herramienta valiosa para la toma de decisiones en un entorno comercial dinámico.

## Data Cleaning

En el proceso de limpieza de datos, se realizaron los siguientes pasos clave:

- **Imputación de Valores Nulos**: Se completaron los valores faltantes en las series temporales utilizando técnicas de imputación específicas de la librería `darts`, asegurando una continuidad en los datos.
- **Codificación y Transformación**: Se transformaron variables categóricas como `store_nbr` y `family` mediante codificación (encoding) para hacerlas compatibles con los modelos.
- **Escalado de Variables**: Se aplicó escalado a las variables numéricas para mejorar el rendimiento y estabilidad del modelo durante el entrenamiento.
- **Generación de Nuevas Características**: Se crearon nuevas variables temporales, como días festivos y atributos de tendencia estacional, para enriquecer los datos y capturar patrones estacionales en las ventas.

## Exploratory Data Analysis (EDA)

Durante la exploración de datos, se realizaron los siguientes análisis:

- **Análisis Estadístico Descriptivo**: Se calculó el rango de fechas, el número de tiendas, familias de productos, y combinaciones únicas de tienda y producto, proporcionando una visión general de la estructura del conjunto de datos.
- **Visualización de Ventas a lo Largo del Tiempo**: Se generaron gráficos de líneas para visualizar tendencias generales y estacionalidad en las ventas de diferentes familias de productos y tiendas.
- **Impacto de las Promociones en las Ventas**: Se exploró la relación entre las promociones (`onpromotion`) y las ventas, analizando cómo afectan las promociones al comportamiento de compra en distintas tiendas y productos.
- **Efecto de Eventos y Festividades**: Se investigó la influencia de eventos especiales y festivos en los picos de ventas, identificando variaciones estacionales y patrones de consumo relacionados con estos eventos.

## Training Preparation

Antes de entrenar el modelo, se llevó a cabo una preparación exhaustiva de los datos para construir una serie de covariables que permitan capturar mejor los patrones temporales en las ventas. Este proceso incluye la creación de series de tiempo específicas para las variables de entrada y la definición de pipelines para transformar los datos de forma eficiente. Los pasos clave fueron los siguientes:

### Creación de Series de Tiempo Objetivo y Covariables

- **Series de Tiempo Objetivo**: Se generaron series de tiempo para las ventas de cada familia de productos en cada tienda, permitiendo que el modelo enfoque las predicciones en cada combinación de tienda y familia.
- **Covariables Futuras**: Se crearon covariables basadas en atributos temporales como el año, mes, día, día de la semana y eventos de calendario (festivos o fechas especiales). También se añadieron medias móviles de los precios del petróleo (7, 14 y 28 días) para capturar patrones de tendencia.
- **Covariables Pasadas**: Se incluyeron transacciones previas como covariables, proporcionando al modelo información sobre el comportamiento histórico relevante para las predicciones.

### Generación de Covariables de Promociones y Festividades

- **Eventos y Festividades**: Se generaron series de tiempo para cada tienda, representando eventos como promociones, días festivos nacionales y eventos locales (e.g., el Día de la Madre). Estos eventos se agregaron como variables binarias indicando si ocurrieron en una fecha específica.
- **Promociones**: Se procesaron con medias móviles (7, 14 y 28 días) para proporcionar al modelo el efecto acumulativo de las promociones en el tiempo.

### Construcción de Pipelines de Transformación

Para facilitar la normalización y transformación de las series de tiempo, se creó una **pipeline de transformación** que incluye pasos para:

- **Rellenar Valores Faltantes**: Uso de `MissingValuesFiller` para garantizar que no haya huecos en las series de tiempo.
- **Transformación Logarítmica**: Aplicada en los datos de ventas para estabilizar la variabilidad y mitigar el impacto de valores atípicos.
- **Escalado de Datos**: Normalización de las series de tiempo con `Scaler` para asegurar consistencia en las variables.
- **Codificación de Covariables Estáticas**: Transformación de variables como el tipo de tienda y ubicación mediante one-hot encoding.

## Training and Evaluation

### Configuración del Modelo

Se establecieron varias combinaciones de parámetros de retrasos (**lags**) para series de tiempo objetivo, covariables futuras y covariables pasadas. Estas configuraciones permiten al modelo capturar patrones de diferentes períodos, desde semanales hasta anuales, considerando factores temporales de corto y largo plazo.

Las configuraciones específicas probadas fueron:

1. **Configuración 1**:
   - `lags`: 63 (retrasos de hasta 63 días para la serie de ventas)
   - `lags_future_covariates`: (14, 1) (covariables futuras con retrasos de 14 días y ventana de 1 día)
   - `lags_past_covariates`: [-16, -17, -18, -19, -20, -21, -22] (retrasos en covariables pasadas como transacciones)

2. **Configuración 2**:
   - `lags`: 7 (retrasos de hasta 7 días para la serie de ventas)
   - `lags_future_covariates`: (16, 1)
   - `lags_past_covariates`: [-16, -17, -18, -19, -20, -21, -22]

3. **Configuración 3**:
   - `lags`: 365 (retrasos de hasta 365 días, capturando patrones anuales)
   - `lags_future_covariates`: (14, 1)
   - `lags_past_covariates`: [-16, -17, -18, -19, -20, -21, -22]

4. **Configuración 4**:
   - `lags`: 730 (retrasos de hasta 730 días, capturando patrones bienales)
   - `lags_future_covariates`: (14, 1)
   - `lags_past_covariates`: [-16, -17, -18, -19, -20, -21, -22]

### Entrenamiento del Modelo

- Para cada combinación de parámetros, se entrenó un modelo `LightGBMModel` específico para cada familia de productos.
- Durante el entrenamiento, se dividieron los datos en conjunto de entrenamiento y validación, asegurando que las predicciones se realizaran sobre datos de validación no vistos.
- Se utilizó la GPU para acelerar el proceso, gracias a la implementación de `LightGBMModel` en `darts`.

### Predicciones y Evaluación de Resultados

- Tras el entrenamiento, se generaron predicciones para el período de validación (16 días). Se aplicaron transformaciones inversas para devolver los valores a su escala original.
- Ajustes en las predicciones para series de ventas recientes sin actividad.

### Promedio de Predicciones

Para reducir la variabilidad, se calculó el promedio de todas las predicciones generadas con las configuraciones de parámetros. Esta técnica de ensamblaje mejoró la estabilidad y precisión de las predicciones finales.

### Visualización de Resultados

Se graficaron las ventas reales y las predicciones para comparar el rendimiento en diferentes tiendas y familias. La gráfica muestra las ventas reales y las predicciones de varias configuraciones, observando el ajuste de las predicciones a los patrones de ventas.

Esta etapa de entrenamiento y evaluación permitió identificar los mejores parámetros y robustecer las predicciones finales, esenciales para una predicción precisa en ventas.
