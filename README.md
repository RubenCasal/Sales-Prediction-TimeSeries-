# Sales-Prediction-TimeSeries

Este proyecto aborda el desafío propuesto por una competencia de Kaggle, donde se busca predecir las ventas de distintas familias de productos en múltiples tiendas de Favorita, una cadena ubicada en Ecuador. Utilizando técnicas de time series, el objetivo es desarrollar un modelo que permita estimar las ventas futuras a partir de datos históricos, lo cual es crucial para mejorar la planificación de inventarios, gestionar promociones y optimizar el suministro de productos.

La solución incluye una serie de modelos predictivos que se entrenan con datos que incorporan factores como promociones, transacciones históricas, fluctuaciones de precios de petróleo y eventos especiales. Este enfoque permite capturar los patrones y tendencias en las ventas, ofreciendo una herramienta valiosa para la toma de decisiones en un entorno comercial dinámico.

### Data Cleaning

En el proceso de limpieza de datos, se realizaron los siguientes pasos clave:

- **Imputación de Valores Nulos**: Se completaron los valores faltantes en las series temporales utilizando técnicas de imputación específicas de la librería `darts`, asegurando una continuidad en los datos.
- **Codificación y Transformación**: Se transformaron variables categóricas como `store_nbr` y `family` mediante codificación (encoding) para hacerlas compatibles con los modelos.
- **Escalado de Variables**: Se aplicó escalado a las variables numéricas para mejorar el rendimiento y estabilidad del modelo durante el entrenamiento.
- **Generación de Nuevas Características **: Se crearon nuevas variables temporales, como días festivos y atributos de tendencia estacional, para enriquecer los datos y capturar patrones estacionales en las ventas.


### Exploratory Data Analysis (EDA)

Durante la exploración de datos, se realizaron los siguientes análisis:

- **Análisis Estadístico Descriptivo**: Se calculó el rango de fechas, el número de tiendas, familias de productos, y combinaciones únicas de tienda y producto, proporcionando una visión general de la estructura del conjunto de datos.
- **Visualización de Ventas a lo Largo del Tiempo**: Se generaron gráficos de líneas para visualizar tendencias generales y estacionalidad en las ventas de diferentes familias de productos y tiendas.
- **Impacto de las Promociones en las Ventas**: Se exploró la relación entre las promociones (`onpromotion`) y las ventas, analizando cómo afectan las promociones al comportamiento de compra en distintas tiendas y productos.
- **Efecto de Eventos y Festividades**: Se investigó la influencia de eventos especiales y festivos en los picos de ventas, identificando variaciones estacionales y patrones de consumo relacionados con estos eventos.


### Training Preparation

Antes de entrenar el modelo, se llevó a cabo una preparación exhaustiva de los datos para construir una serie de covariables que permitan capturar mejor los patrones temporales en las ventas. Este proceso incluye la creación de series de tiempo específicas para las variables de entrada y la definición de pipelines para transformar los datos de forma eficiente. Los pasos clave fueron los siguientes:

- **Creación de Series de Tiempo Objetivo y Covariables**:
   - Se generaron **series de tiempo objetivo** para las ventas de cada familia de productos en cada tienda. Esto permite que el modelo enfoque las predicciones específicamente en cada combinación de tienda y familia.
   - Se crearon **covariables futuras** basadas en atributos temporales, como el año, mes, día, día de la semana y eventos de calendario (como festivos o fechas especiales). También se añadieron medias móviles de los precios del petróleo (7, 14 y 28 días) para capturar patrones de tendencia que puedan influir en las ventas.
   - Además, se añadieron **covariables pasadas** que incluyen las transacciones previas, proporcionando al modelo información sobre el comportamiento histórico que puede ser relevante para las predicciones.

- **Generación de Covariables de Promociones y Festividades**:
   - Se generaron series de tiempo adicionales para cada tienda, representando eventos como promociones, días festivos nacionales, eventos locales y otros eventos especiales (e.g., el Día de la Madre o eventos de fútbol). Estos eventos se agregaron como variables binarias que indican si un evento específico tuvo lugar en una fecha determinada.
   - Las promociones se procesaron con medias móviles (7, 14 y 28 días) para suavizar la información y proporcionar al modelo una idea del efecto acumulativo de las promociones en el tiempo.

- **Construcción de Pipelines de Transformación**:
   - Para facilitar la normalización y la transformación de las diferentes series de tiempo, se creó una **pipeline de transformación** que incluye pasos para:
      - **Rellenar Valores Faltantes**: Uso de `MissingValuesFiller` para garantizar que las series de tiempo no tengan huecos, permitiendo que el modelo trabaje con datos continuos.
      - **Transformación Logarítmica**: Se aplica una transformación logarítmica en los datos de ventas para estabilizar la variabilidad y mitigar el impacto de valores atípicos.
      - **Escalado de Datos**: Se normalizan las series de tiempo con `Scaler` para asegurar que las variables tengan una escala consistente, mejorando la estabilidad y el rendimiento del modelo.
      - **Codificación de Covariables Estáticas**: Las covariables estáticas como el tipo de tienda y la ubicación (ciudad, estado) se transformaron mediante one-hot encoding, lo cual permite que el modelo las interprete de forma adecuada durante el entrenamiento.

### Entrenamiento y Evaluación del Modelo (Training and Model Evaluation)

En esta sección, se definieron y evaluaron diferentes configuraciones de parámetros para entrenar modelos de predicción de ventas utilizando `LightGBMModel`. A continuación, se describe el proceso detallado:

- **Definición de Configuraciones de Parámetros**:
   - Se establecieron varias combinaciones de parámetros de retrasos (**lags**) para series de tiempo objetivo, covariables futuras y covariables pasadas. Estas configuraciones permiten al modelo capturar patrones de diferentes períodos, desde semanales hasta anuales, considerando factores temporales de corto y largo plazo.

   - Las configuraciones probadas incluyeron:
     - Retrasos diarios de hasta 63 días.
     - Uso de covariables futuras con retrasos de 14 a 16 días.
     - Retrasos en covariables pasadas como transacciones históricas para capturar tendencias estacionales en las ventas.

- **Entrenamiento del Modelo**:
   - Para cada combinación de parámetros, se entrenó un modelo `LightGBMModel` específico para cada familia de productos.
   - Durante el entrenamiento, se dividieron los datos en conjunto de entrenamiento y validación, asegurando que las predicciones se realizaran sobre datos de validación que el modelo no había visto previamente.
   - Se utilizó la GPU para acelerar el proceso de entrenamiento, gracias a la implementación de `LightGBMModel` en `darts` que permite la ejecución en dispositivos compatibles.

- **Predicciones y Evaluación de Resultados**:
   - Una vez entrenados los modelos, se generaron predicciones para el período de validación (16 días). Las predicciones se almacenaron para cada configuración de parámetros y se realizaron transformaciones inversas para devolver los valores a su escala original.
   - Las predicciones fueron ajustadas en casos donde las ventas pasadas recientes eran cero, para reflejar mejor la falta de actividad de ventas en dichos períodos.

- **Promedio de Predicciones**:
   - Para reducir la variabilidad de las predicciones individuales, se calculó el promedio de todas las predicciones generadas con distintas configuraciones de parámetros. Esta técnica de ensamblaje permitió mejorar la estabilidad y precisión de las predicciones finales.

- **Visualización de Resultados**:
   - Se graficaron las ventas reales y las predicciones para comparar visualmente el rendimiento del modelo en diferentes tiendas y familias de productos. La gráfica adjunta muestra las ventas reales (línea azul) y las predicciones de distintas configuraciones (líneas punteadas) para algunas tiendas y familias seleccionadas, permitiendo observar el ajuste de las predicciones a los patrones de ventas.
   
Esta etapa de entrenamiento y evaluación permitió identificar los mejores parámetros para el modelo, además de aplicar técnicas de ensamblaje para robustecer las predicciones finales, lo cual es fundamental en el contexto de predicción de ventas.




