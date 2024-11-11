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





