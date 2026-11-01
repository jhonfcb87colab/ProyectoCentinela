# Propuesta Resumen: Proyecto Centinela (Fase 1)
**Línea Base con Perceptrón Multicapa (MLP) para la Predicción de Producción Frutícola**  
*Profundización I: Redes Neuronales y Deep Learning*
*Postgrado: Maestría en Ciencia de Datos (USTA)*  
*Presentado:* `John Fernando Cruz Becerra`  

---

## 1. Definición del Problema y Enfoque de Negocio

El **Proyecto Centinela** busca resolver un problema clave en la gestión agrícola del departamento del Valle del Cauca: la estimación del volumen de producción agrícola (expresado en toneladas) según combinaciones específicas de municipio, subregión, tipo de cultivo, ciclo productivo y año.

* **Cliente objetivo:** Secretaría de Agricultura y Desarrollo Rural del Valle del Cauca, así como asociaciones de productores locales.
* **Objetivo de negocio:** Contar con una herramienta analítica preventiva para optimizar la planificación logística, la asignación de insumos públicos y la detección temprana de anomalías en el rendimiento agrícola.
* **Unidad de análisis:** Registro consolidado anual por combinación `Municipio` – `Cultivo` – `Ciclo` – `Año`.
* **Variable objetivo ($y$):** `produccion_toneladas` (variable continua $\ge 0$). Al tratarse de un problema de **regresión supervisada**, la neurona de salida no utiliza funciones de compresión (como sigmoide o softmax), sino una activación lineal directa.

---

## 2. Pipeline de Procesamiento de Datos y Vectorización

El procesamiento de los datos de origen público (portal `datos.gov.co` vía API Socrata) sigue un flujo estricto de ingeniería de características para alimentar adecuadamente la red en PyTorch:

```
[API Socrata JSON] ➔ [Pandas DataFrame] ➔ [Depuración NaN & Metadatos] ➔ [Split Train/Val/Test] ➔ [ColumnTransformer] ➔ [Tensores PyTorch (88 Dims)]
```

### Detalle de las Etapas:
1. **Ingesta y Limpieza:**
   * Se descargaron **9.992 registros**.
   * Se eliminaron metadatos internos de la plataforma (columnas con prefijo `:`).
   * Se aplicó conversión numérica estricta (`errors='coerce'`) y filtrado de registros sin etiqueta de producción conocida.
2. **Estrategia de Partición (Reproducibilidad con `SEED = 42`):**
   * **Entrenamiento (Train 70%):** 6.994 registros para el ajuste de pesos.
   * **Validación (Val 15%):** 1.499 registros para monitorear el ajuste y comparar hiperparámetros.
   * **Prueba (Test 15%):** 1.499 registros reservados exclusivamente para la evaluación final.
3. **Transformación Vectorial (`ColumnTransformer`):**
   * **Variables Categóricas:** `municipio`, `subregion`, `cultivo` y `ciclo` transformadas mediante *One-Hot Encoding* (`sparse_output=False`, `handle_unknown='ignore'`).
   * **Variables Numéricas:** `anio` escalada mediante *StandardScaler*.
   * **Dimensión de Entrada Resultante:** Vector denso de **88 características** ($X \in \mathbb{R}^{n \times 88}$).

---

## 3. Arquitectura de la Red Neuronal (Perceptrón Multicapa)

Se diseñó la clase `MLPFrutales` en PyTorch, compuesta por 3 capas ocultas densas con activaciones no lineales **ReLU** y regularización por **Dropout**.

```
Entrada (88) ──> [Linear 128 + ReLU + Dropout] ──> [Linear 64 + ReLU + Dropout] ──> [Linear 32 + ReLU + Dropout] ──> Salida [Linear 1]
```

### Fórmulas Matemáticas de la Red:
$$h_1 = \text{ReLU}(W_1 x + b_1), \quad W_1 \in \mathbb{R}^{128 \times 88}$$
$$h_2 = \text{ReLU}(W_2 h_1 + b_2), \quad W_2 \in \mathbb{R}^{64 \times 128}$$
$$h_3 = \text{ReLU}(W_3 h_2 + b_3), \quad W_3 \in \mathbb{R}^{32 \times 64}$$
$$\hat{y} = W_4 h_3 + b_4, \quad W_4 \in \mathbb{R}^{1 \times 32}$$

---

## 4. Estrategia de Entrenamiento y Experimentación

Se implementó un bucle explícito de entrenamiento en PyTorch ejecutado durante **50 épocas** sobre una GPU CUDA, utilizando `TensorDataset` y `DataLoader` con un tamaño de lote (*batch size*) de **32**.

* **Función de Pérdida:** Error Cuadrático Medio ($\text{MSE}$).
  $$\text{MSE} = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$
* **Optimizador:** Adam con tasa de aprendizaje $\alpha = 0.001$.
* **Secuencia por Lote:** `zero_grad()` $\to$ `forward` $\to$ `loss.backward()` $\to$ `optimizer.step()`.

### Experimentos Comparativos:
Se entrenaron dos variantes para analizar el impacto de la regularización frente al sobreajuste:
1. **Modelo Base:** Sin regularización (`dropout_rate = 0.0`).
2. **Modelo Regularizado:** Con apagado aleatorio del 20% de neuronas (`dropout_rate = 0.2`).

#### Evolución de la Pérdida ($\text{MSE}$) a lo Largo del Entrenamiento:

| Época | Pérdida Train (Base) | Pérdida Val (Base) | Pérdida Train (Reg 0.2) | Pérdida Val (Reg 0.2) |
| :---: | :---: | :---: | :---: | :---: |
| **1** | 27.292.381,82 | 27.989.608,56 | 27.380.064,10 | 28.174.427,75 |
| **10** | 15.628.376,24 | 18.338.578,18 | 16.176.615,47 | 18.637.689,03 |
| **20** | 14.323.952,23 | 17.171.327,56 | 15.023.855,43 | 17.403.162,76 |
| **30** | 13.343.826,47 | 16.305.608,58 | 14.469.354,42 | 16.645.469,87 |
| **40** | 12.386.648,27 | 15.395.319,57 | 13.369.308,71 | 15.953.176,02 |
| **50** | 11.228.338,27 | 14.407.652,02 | 12.346.210,04 | 15.224.999,45 |

*Observación:* Ambas configuraciones muestran una convergencia constante. La бреча entre la pérdida de entrenamiento y la de validación evidencia la necesidad de incorporar variables exógenas más explicativas en etapas posteriores.

---

## 5. Evaluación del Desempeño en Test

La evaluación final del modelo regularizado sobre los **1.499 registros no vistos** del conjunto de prueba arrojó las siguientes métricas globales:

$$\text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2} = 4.442,53 \text{ toneladas}$$
$$\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - \hat{y}_i| = 1.368,27 \text{ toneladas}$$
$$R^2 = 1 - \frac{\sum (y_i - \hat{y}_i)^2}{\sum (y_i - \bar{y})^2} = 0,4606$$

### Interpretación Analítica:
* **$R^2 = 0,4606$:** El Perceptrón Multicapa logra explicar aproximadamente el **46,06% de la variabilidad** en la producción agrícola utilizando únicamente las variables categóricas territoriales y el año.
* **MAE (1.368,27 t):** Representa el margen medio de error absoluto por predicción.
* **Sensibilidad del RMSE (4.442,53 t):** Al ser sustancialmente mayor que el MAE, indica que la red enfrenta penalizaciones elevadas debido a valores atípicos o picos de alta producción en ciertos municipios clave.

---

## 6. Auditoría Ética e Impacto Territorial

En concordancia con los marcos de Inteligencia Artificial Responsable, se identifican los siguientes riesgos de decisión asociados a los errores del modelo:

1. **Riesgo por Subestimación (Falsos Negativos de Producción):**
   Predecir un volumen inferior al real puede ocasionar falta de capacidad logística (bodegaje, transporte refrigerado), generando pérdidas de cosecha para los pequeños agricultores.
2. **Riesgo por Sobreestimación (Falsos Positivos de Producción):**
   Generar estimaciones excesivamente optimistas puede provocar caídas artificiales en los precios de compra locales o sobrecostos en planes gubernamentales de abastecimiento.
3. **Mitigación:** El modelo debe emplearse como un **sistema de soporte a la decisión**, complementado por inspecciones agronómicas en campo y sin automatizar compras ni asignaciones presupuestales de forma ciega.

---

## 7. Conclusiones y Transición a la Fase 2

1. **Línea Base Sólida:** El MLP desarrollado constituye una línea base funcional y reproducible en PyTorch para datos tabulares agrícolas.
2. **Necesidad Multimodal (Fase 2):** Para superar el techo del 46% en el coeficiente $R^2$, es imprescindible enriquecer la representación tabular mediante:
   * **Imágenes Satelitales:** Índices de vegetación (NDVI/EVI) provenientes de Sentinel-2.
   * **Series Climatológicas:** Series temporales de precipitación y temperatura de ERA5-Land.