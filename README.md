 # Taller #3: Limpieza, Ingeniería y Validación de Datos 📊

## 📋 Información del Proyecto
* **Profesora:** Adriana Collaguazo
* **Integrantes:**
  * Ritter Briones
  * Mariana Mora
  * Anthony Polo


##  🎯 Objetivo del Análisis
El objetivo principal de este taller es aplicar técnicas de preprocesamiento, limpieza, escalado y validación automática de datos sobre el historial de ventas de una cafetería. Para ello, se busca mejorar la calidad de la información mediante el tratamiento de datos faltantes a través de su eliminación o imputación, la corrección de inconsistencias en los tipos de datos y de valores atípicos, así como el escalado y la normalización de las variables numéricas. Estas actividades permitirán obtener un conjunto de datos confiable, consistente y preparado para su uso en procesos de analítica avanzada y modelos de Machine Learning.



## 🗄️ Dataset Utilizado
* **Nombre:** `dirty_cafe_sales.csv` ("Tech Coffee Shop")
* **Cantidad de registro inicial:** 10,000 registros.
* **Estado inicial del Dataset:**
  * **Problemas de Tipo:** Todas las variables fueron inicialmente interpretadas como tipo `object` (cadena de texto).
  * **Datos Faltantes:** Alta concentración de valores nulos, especialmente en las columnas `Location` y `Payment Method` (identificado mediante gráficos de calor con la librería `missingno`).
  * **Valores Corruptos:** Presencia de cadenas como `'ERROR'` y `'UNKNOWN'` en variables numéricas clave como `Total Spent`.



## 🛠️ Metodología y Estrategias Aplicadas

1. **Exploración y Perfilado:** Uso de `ydata-profiling` (`ProfileReport`) para auditoría automatizada y `missingno` para mapear patrones de ausencia de datos.
2. **Tratamiento de Nulos Críticos:** * Se evaluó la eliminación de filas y columnas con nulos, determinando que **no eran viables** por la pérdida masiva de información (más del 5% en filas y pérdida casi total de columnas).
   * Se transformaron los errores de texto a NaN (`pd.to_numeric(..., errors='coerce')`).
   
3. **Escalado y Normalización:** Se experimentó con múltiples transformaciones para la variable de precio:
   * **MinMaxScaler:** Ajuste al rango $[0, 1]$.
   * **RobustScaler:** Manejo robusto basado en cuantiles para mitigar el impacto de valores atípicos.
   * **PowerTransformer (Box-Cox):** Estabilización de la varianza y aproximación a una distribución normal.
   * **StandardScaler:** Estandarización clásica ($\mu = 0, \sigma = 1$).
   * **Normalizer (L2):** Normalización por filas aplicada a la interacción entre cantidad y precio.
4. **Validación con Pandera:** Creación de un esquema robusto (`DataFrameSchema`) para auditar de forma automatizada las reglas de negocio (IDs únicos, no negatividad de montos y precios mayores a cero).


## 📌 Resumen de Hallazgos y Conclusiones

* **Incompatibilidad de Eliminación Directa:** Eliminar registros nulos a ciegas en este dataset destruía la integridad de la muestra (dejando solo la columna `Transaction ID` en el caso de la eliminación por columnas), lo que demostró la necesidad estricta de aplicar **estrategias de imputación**.
* **El Peligro del 'Backward Fill' (`bfill`):** Al inspeccionar los primeros 100 registros tras aplicar `bfill` en `Total Spent`, se evidenciaron **inconsistencias matemáticas** en filas donde el precio unitario y la cantidad no multiplicaban el total imputado. Esto resalta que la imputación ciega puede alterar la lógica del negocio.
* **Sensibilidad al Escalado:** El precio unitario presentaba distribuciones sesgadas que requirieron la evaluación de `RobustScaler` para no distorsionar el comportamiento real de las compras debido a anomalías.
* **Gobernanza de Datos:** La validación automatizada mediante un esquema de `Pandera` (`schema.validate`) permitió capturar de forma "perezosa" (`lazy=True`) todos los casos de falla concurrentes, asegurando que cualquier tubería de datos futura rechace registros que no cumplan con los criterios de calidad mínimos establecidos.
