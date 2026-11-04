# Bitácora de IA · Proyecto Centinela

La bitácora se diligencia a lo largo del proyecto, no al final. No se registra cada indicación (*prompt*): se documentan las interacciones que cambiaron una decisión técnica (arquitectura, datos, métricas, un error resuelto, un enfoque descartado). Cada entrada lleva la fecha y la hora de la consulta, anotadas cuando ocurre.

A un asistente de IA no se le pegan ni se le suben registros con datos personales ni datos de terceros sin su permiso: se le muestran el esquema y filas sintéticas, y la bitácora registra qué datos se compartieron.

## 1. Datos de la entrega

| Campo | Detalle |
| --- | --- |
| **Integrantes** | John Fernando Cruz Becerra (entrega individual) |
| **Proyecto** | Centinela · alerta temprana de caída de producción frutícola en el Valle del Cauca (clasificación binaria con MLP sobre datos tabulares de producción) |
| **Fase** | Fase 1 · MLP |
| **Primer identificador de esta fase** | E-1 |
| **Fecha de entrega** | 2026-11-04 |

## 2. Declaración de uso de IA

### Declaración de uso (la firma cada integrante)

Quienes firman declaran que durante esta fase usaron las siguientes herramientas de IA generativa:

- **Asistente de IA generativa (herramienta y modelo no registrados en el cuaderno · sesión de la v1 y de la revisión de la v2)** → para formular el bucle de entrenamiento en PyTorch, construir la etiqueta binaria sin fuga de información, diseñar la comparación contra la línea base y validar el cuaderno contra la rúbrica de la actividad.
- **Asistente de IA generativa (misma herramienta · revisión posterior de la v2)** → para auditar el cuaderno contra la rúbrica del docente y detectar errores metodológicos (umbral barrido sobre prueba, rezagos con NaN→0, regularización inestable).

Partes del entregable redactadas o programadas sin IA: la propuesta institucional (`04Propuesta_Proyecto_Centinela.txt`), el marco ético (sección 13), la interpretación honesta de los resultados (sección 15), la verificación manual de la licencia CC BY-SA 4.0 en los metadatos de datos.gov.co y el checklist de la Ley 1581 de 2012 (celdas 5–7 del cuaderno).

Quienes firman declaran también que **comprenden, verificaron y pueden defender** como propio cada artefacto entregado, que cada integrante responde en la sustentación por una parte distinta del trabajo y que esta bitácora refleja con honestidad cómo se usaron estas herramientas.

| Integrante | Firma | Fecha |
| --- | --- | --- |
| John Fernando Cruz Becerra | _______JHONFCB_____ | 2026-11-04 |

### Declaración de no uso (alternativa)

*(No aplica: se usó IA generativa en esta fase.)*

| Integrante | Firma | Fecha |
| --- | --- | --- |
| _(nombre)_ | ______________________ | _(AAAA-MM-DD)_ |

En equipo, cada integrante firma la declaración que le corresponde: la de uso o la de no uso. Las filas que sobran se borran.

## 3. Registro de interacciones

Cada interacción se marca como **Aceptada · Modificada · Rechazada**. La evidencia de origen (la salida literal de la IA, o una captura de pantalla con fecha y herramienta) es obligatoria en las entradas Modificada y Rechazada; en las Aceptadas es opcional.

**Cómo se numera.** La ficha E-0 es el molde: se copia una vez por interacción y cada copia recibe el identificador siguiente (E-1, E-2...). La numeración no se reinicia entre fases: la Fase 2 continúa desde el último identificador de la Fase 1. El mismo identificador se escribe como comentario en la línea del código que cambió (`# E-3`), para que la entrada se pueda localizar en el cuaderno.

**Cuántas entradas.** Para llegar a Básico, el Criterio 5 pide al menos tres entradas fechadas, con su decisión y una justificación en cada una.

### E-1

| Campo | Registro |
| --- | --- |
| **ID** | E-1 |
| **Fecha y hora** | 2026-11-03 (v1; hora exacta no registrada en el cuaderno, sesión previa al sello del archivo `02Solucion..._03.ipynb`, 21:35) |
| **Fase · paso** | F1 · Paso 1 (formulación del objetivo y del modelo) |
| **Integrante que hizo la consulta** | John Fernando Cruz Becerra (entrega individual, se omite) |
| **Herramienta y modelo** | Asistente de IA generativa · modelo no registrado en el cuaderno |
| **Celda o *commit* afectado** | v1 completa; en la v2, las celdas 12–16, 29–33 y 37–40 (`# E-1` en la celda 13, construcción de la etiqueta) |
| **Objetivo de la consulta** | Construir un modelo que anticipara la producción agrícola para la Secretaría de Agricultura del Valle del Cauca |
| **Indicación (resumen)** | «Entrena un MLP para predecir la producción en toneladas con municipio, cultivo y año» |
| **Qué devolvió la IA** | Regresión continua con `MSELoss`, salida lineal y partición aleatoria 70/15/15 |
| **Datos compartidos con la IA** | Ninguno: solo se describió el esquema del problema (municipio, cultivo, año → toneladas) sin filas reales |
| **Decisión** | Rechazada |
| **Justificación** | La regresión continua no responde a la pregunta de negocio (¿habrá caída?): el MSE optimiza el ajuste medio de toneladas y no penaliza de forma prioritaria las caídas; además, la partición aleatoria sitúa años vecinos de la misma serie a ambos lados del corte (fuga temporal), inflando las métricas. La v2 corrige con clasificación binaria (umbral 80 % de la mediana de los 5 años anteriores) y partición cronológica estricta. |
| **Cómo se verificó** | Reproducción del fallo en la celda 38 del cuaderno: con partición aleatoria, el 87,0 % de las filas de prueba tienen un año vecino de su serie en entrenamiento; la regresión logística con la misma semilla (42) pasa de recall 0,731 (aleatoria) a 0,709 (cronológica): la ventaja ilusoria de la v1 era 0,022–0,067 en las métricas reportadas. |
| **Evidencia de origen** | Sección 14 del cuaderno (fila de la v1: «Regresión continua con `MSELoss`, salida lineal y partición aleatoria 70/15/15») y celda 38 (reproducción con código y salida impresa) |

### E-2

| Campo | Registro |
| --- | --- |
| **ID** | E-2 |
| **Fecha y hora** | 2026-11-03–04 (transición v1→v2; hora exacta no registrada en el cuaderno) |
| **Fase · paso** | F1 · Paso 2 (variable objetivo binaria) |
| **Integrante que hizo la consulta** | John Fernando Cruz Becerra (entrega individual, se omite) |
| **Herramienta y modelo** | Asistente de IA generativa · modelo no registrado en el cuaderno |
| **Celda o *commit* afectado** | Celda 13 del cuaderno (`# E-2`, construcción de `mediana_5_prev`, `y`, rezagos y prueba de no filtración) |
| **Objetivo de la consulta** | Construir la etiqueta binaria de «caída» con los 5 años anteriores, sin que la producción del año a predecir contamine sus propios predictores |
| **Indicación (resumen)** | «¿Cómo construyo la etiqueta de caída sin fuga de información?» |
| **Qué devolvió la IA** | Propuso `shift(1).rolling(5).median()` por serie `(id_municipio, cultivo, ciclo)` antes de comparar con el 80 % |
| **Datos compartidos con la IA** | Ninguno: solo el esquema de columnas (`id_municipio`, `cultivo`, `ciclo`, `anio`, `produccion_toneladas`) y sus tipos |
| **Decisión** | Aceptada |
| **Justificación** | Cumple la restricción del docente: `shift(1)` desplaza antes de la ventana móvil, de modo que la mediana de `t` solo usa `t−1 … t−5` (teoría del curso: sin filtración en series temporales; los rezagos y transformaciones no miran el futuro). |
| **Cómo se verificó** | Prueba automática de no filtración en la celda 13: en 200 muestras aleatorias (semilla 42) la mediana coincide con la mediana de las 5 producciones anteriores de la serie → «OK»; además se verifica que ninguna columna con la producción contemporánea entra en `X` y que `lag1`–`lag3` nunca son NaN en filas etiquetables. |
| **Evidencia de origen** | Opcional en Aceptada: la celda 13 del cuaderno contiene el código y la salida («(a) Mediana calculada solo con los 5 años anteriores en 200 muestras: OK») |

### E-3

| Campo | Registro |
| --- | --- |
| **ID** | E-3 |
| **Fecha y hora** | 2026-11-03–04 (transición v1→v2; hora exacta no registrada en el cuaderno) |
| **Fase · paso** | F1 · Paso 8 (bucle de entrenamiento) |
| **Integrante que hizo la consulta** | John Fernando Cruz Becerra (entrega individual, se omite) |
| **Herramienta y modelo** | Asistente de IA generativa · modelo no registrado en el cuaderno |
| **Celda o *commit* afectado** | Celdas 22 y 24 del cuaderno (`# E-3`, clase `MLPCentinela` y función `entrenar_modelo`) |
| **Objetivo de la consulta** | Escribir el bucle de entrenamiento en PyTorch para clasificación binaria con `BCEWithLogitsLoss` |
| **Indicación (resumen)** | «Escribe el bucle de entrenamiento en PyTorch para clasificación binaria» |
| **Qué devolvió la IA** | Un bucle con `nn.Sigmoid()` dentro del modelo y `BCEWithLogitsLoss` como pérdida, es decir, sigmoide y pérdida con logits a la vez |
| **Datos compartidos con la IA** | Ninguno: solo el esquema del problema (entrada tabular, salida binaria) |
| **Decisión** | Modificada |
| **Justificación** | Antipatrón de la doble sigmoide: `BCEWithLogitsLoss` ya aplica la sigmoide internamente sobre el logit (numéricamente estable); poner `nn.Sigmoid()` en el `forward` y luego la pérdida con logits aplica la función dos veces y degrada la señal de gradiente. Se conservó la salida como logit y `torch.sigmoid` se aplica solo en evaluación (celdas 22 y 24). |
| **Cómo se verificó** | La arquitectura imprime una salida `Linear(in_features=32, out_features=1, bias=True)` sin activación; las pérdidas de entrenamiento y validación convergen con las curvas de la celda 27 (brecha val−train final del regularizado: −0,005), lo que es coherente con el uso correcto de la pérdida con logits. |
| **Evidencia de origen** | Salida impresa del modelo en la celda 22 (sin sigmoide en el `forward`): `MLPCentinela(... (9) Linear(in_features=32, out_features=1, bias=True))` |

### E-4

| Campo | Registro |
| --- | --- |
| **ID** | E-4 |
| **Fecha y hora** | 2026-11-03–04 (transición v1→v2; hora exacta no registrada en el cuaderno) |
| **Fase · paso** | F1 · Paso 6 (líneas base) |
| **Integrante que hizo la consulta** | John Fernando Cruz Becerra (entrega individual, se omite) |
| **Herramienta y modelo** | Asistente de IA generativa · modelo no registrado en el cuaderno |
| **Celda o *commit* afectado** | Celdas 17–18 y 31 del cuaderno (`# E-4`, métricas de línea base y barrido de umbral) |
| **Objetivo de la consulta** | Comparar el MLP contra una línea base para valorar si la red aporta algo |
| **Indicación (resumen)** | «¿Cómo comparo el MLP con una línea base?» |
| **Qué devolvió la IA** | Proponía comparar solo contra la clase mayoritaria (alertar siempre «sin caída») |
| **Datos compartidos con la IA** | Ninguno: solo la pregunta metodológica |
| **Decisión** | Modificada |
| **Justificación** | La clase mayoritaria es una referencia trivial que no puede alertar ninguna caída (recall 0); la Secretaría necesita saber si el modelo supera la regla que ya podría aplicar sin modelo: la **persistencia** (alertar en `t` si `t−1` fue caída), línea base exigida por el docente. Además se añadió la selección del umbral operativo por costo FN:FP = 3:1, para responder al argumento ético. |
| **Cómo se verificó** | Tabla de líneas base de la celda 18 con Accuracy y Recall en train/val/test, y barrido de umbral en validación (celda 31): umbral operativo 0,55 (menor costo en validación) aplicado una sola vez sobre prueba; en prueba el MLP supera a la persistencia solo en recall (0,683 vs 0,612 con umbral 0,5) y no en exactitud ni F1 (celdas 30 y 40). |
| **Evidencia de origen** | Salidas de las celdas 18 y 40 del cuaderno con las tablas de métricas de persistencia y MLP |

### E-5

| Campo | Registro |
| --- | --- |
| **ID** | E-5 |
| **Fecha y hora** | 2026-11-03–04 (transición v1→v2; hora exacta no registrada en el cuaderno) |
| **Fase · paso** | F1 · Paso 11 (evaluación en prueba) |
| **Integrante que hizo la consulta** | John Fernando Cruz Becerra (entrega individual, se omite) |
| **Herramienta y modelo** | Asistente de IA generativa · modelo no registrado en el cuaderno |
| **Celda o *commit* afectado** | Celdas 30 y 40 del cuaderno (`# E-5`, métricas de prueba y conclusión honesta) |
| **Objetivo de la consulta** | Generar la tabla de métricas del modelo para el informe de resultados |
| **Indicación (resumen)** | «Genera la tabla de métricas de prueba» |
| **Qué devolvió la IA** | Presentó Accuracy 0,97 sin especificar sobre qué conjunto ni con qué umbral |
| **Datos compartidos con la IA** | Ninguno: fue una consulta sobre cómo reportar, sin filas reales |
| **Decisión** | Rechazada |
| **Justificación** | Cifra no reproducible: un Accuracy 0,97 es incompatible con la prevalencia y el balance reales del problema (29,3 % de caídas) y no se respalda en ninguna ejecución; además el reporte debe declarar el conjunto (2021–2022) y el umbral. Se recalcularon todas las métricas sobre prueba con la semilla fija 42 y se reportaron Accuracy, Recall, Precisión, F1, matriz de confusión y AUC. |
| **Cómo se verificó** | Celda 30: en prueba (n = 764) el MLP con umbral 0,5 obtiene Accuracy 0,716, recall 0,683, precisión 0,581, F1 0,628 y AUC 0,767; el resumen final (celda 41) y la conclusión honesta (celda 40) se generaron con estas cifras de ejecución, no con la cifra de la IA. |
| **Evidencia de origen** | Filas de la sección 14 del cuaderno («Presentó Accuracy 0,97 sin especificar el conjunto», «Cifra no reproducible») y tabla de la celda 30 |

### E-6

| Campo | Registro |
| --- | --- |
| **ID** | E-6 |
| **Fecha y hora** | 2026-11-04 (revisión de la v2; el cuaderno tiene el sello de la descarga de datos a las 04:10:31 UTC) |
| **Fase · paso** | F1 · Revisión final contra la rúbrica |
| **Integrante que hizo la consulta** | John Fernando Cruz Becerra (entrega individual, se omite) |
| **Herramienta y modelo** | Asistente de IA generativa · modelo no registrado en el cuaderno |
| **Celda o *commit* afectado** | Celdas 20, 26–28, 31, 40 del cuaderno (`# E-6`, rezagos sobre la serie completa, regularización y umbral operativo) |
| **Objetivo de la consulta** | Validar el cuaderno de la v2 contra la rúbrica de la actividad y la retroalimentación del docente antes de entregar |
| **Indicación (resumen)** | «Valida el cuaderno contra la rúbrica de la actividad» |
| **Qué devolvió la IA** | Detectó: (1) umbral operativo elegido barriendo sobre prueba; (2) rezagos calculados tras filtrar las filas etiquetables (NaN→0 artificial); (3) umbral que contradecía el argumento ético (no orientado al recall); (4) regularización que no estabilizaba la validación; (5) conclusión que no reconocía que el MLP no supera a la persistencia en exactitud ni F1 |
| **Datos compartidos con la IA** | Solo el esquema del cuaderno y las salidas agregadas de métricas; ninguna fila con datos personales (la base no los contiene) |
| **Decisión** | Aceptada y verificada |
| **Justificación** | Los cinco hallazgos coinciden con la metodología del curso: el umbral debe elegirse solo con validación (celda 31, costo FN:FP = 3:1 → umbral 0,55); los rezagos deben calcularse sobre la serie completa antes de filtrar para que 2005–2007 usen la producción real de 2000–2004 (celda 13 y 20); la regularización se elige por estabilidad de validación (Dropout 0,5, red 32-16, wd 1e-3, celda 26); y la conclusión debe declara explícitamente en qué métricas el MLP pierde frente a la persistencia (celda 40). |
| **Cómo se verificó** | Todas las correcciones se ejecutaron y verificaron en el cuaderno: barrido de umbral solo en validación (celda 31), `shift(k)` sobre la serie completa con assert de rezagos no NaN (celda 13), curvas de la celda 28 con rebote de validación 99 % menor y brecha val−train −0,005, y la conclusión de la celda 40: «el MLP NO supera a la persistencia en exactitud ni en F1». |
| **Evidencia de origen** | Sección 14 del cuaderno, tabla «Revisión posterior de la v2», fila «Aceptada y verificada» con los cinco hallazgos listados |

Para cada interacción nueva se copia la ficha E-0 completa y se cambia su identificador. La ficha E-0 puede quedarse como molde vacío o borrarse: no cuenta como entrada.

## 4. Auditoría a la IA

Obligatoria: se documenta al menos un caso en que la IA falló durante el propio proyecto y cómo se corrigió. Suele ser una de las entradas Modificada o Rechazada de la sección 3, desarrollada a fondo. Los casos que el curso ya publicó (doble sigmoide, bucle sin limpieza de gradientes, fuga del escalador) solo cuentan si ocurrieron en el propio proceso.

**Tipo de auditoría** (se marca una):

- [x] **Real:** la IA produjo el fallo durante el proyecto (la v1 completa: regresión continua + partición aleatoria con fuga temporal).
- [ ] **Inducida y declarada:** la IA no produjo un fallo genuino; se le pidió revisar una parte del propio código (p. ej., el bucle o la evaluación) y se auditó su respuesta. Llega como máximo al nivel Autónomo del Criterio 5.

**Qué evidencia corresponde a cada nivel**

| Nivel del Criterio 5 | Qué muestra la auditoría |
| --- | --- |
| **Básico** | La auditoría describe un fallo de la IA, aunque sea de forma genérica (sin la salida literal ni su ubicación). |
| **Autónomo** | La auditoría documenta un fallo real del propio proyecto, con su **evidencia de origen** (salida literal de la IA, o captura de pantalla con fecha y herramienta), la **corrección** y su **fundamento**. Es también el techo de la auditoría inducida y declarada. |
| **Estratégico** | Además, la auditoría **reproduce** el fallo: una celda ejecuta la versión de la IA sobre los datos propios y la misma semilla, y otra ejecuta la versión corregida con la misma medición, que incluye una métrica de decisión (*recall*, matriz o AUC). |

Lo que no es una auditoría: un error propio que la IA corrigió (se registra en la sección 3 como entrada Aceptada, pero no audita a la IA) y un caso que el curso ya publicó, narrado sin evidencia propia (queda como descripción genérica).

### Ficha del caso

| Campo | Registro |
| --- | --- |
| **Entrada de la bitácora** | E-1 |
| **Contexto** | En la formulación de la v1 del proyecto se preguntó a la IA cómo construir un modelo que anticipara la producción agrícola para la Secretaría de Agricultura del Valle del Cauca (paso 1 de la Fase 1) |
| **Qué dijo la IA** | Entrenar una regresión continua de toneladas con `MSELoss`, salida lineal y partición aleatoria 70/15/15 |
| **Evidencia de origen** | Sección 14 del cuaderno (fila E-1 de la v1: «Regresión continua con `MSELoss`, salida lineal y partición aleatoria 70/15/15») y celda 38 con la reproducción ejecutada |
| **Por qué estaba mal** | (1) La pregunta de negocio es binaria («¿la producción caerá por debajo del 80 % de su nivel habitual?»): la regresión optimiza el error cuadrático medio sobre toneladas, que no prioriza detectar las caídas, único evento que exige una decisión de la Secretaría. (2) La partición aleatoria rompe la estructura temporal: años vecinos de la misma serie `(id_municipio, cultivo, ciclo)` quedan a ambos lados del corte, y los rezagos/la mediana de prueba se «aprenden» de observaciones futuras, inflando las métricas (fuga temporal). |
| **Cómo se detectó** | (1) Retroalimentación del docente (el objetivo debe ser clasificación binaria y está prohibida la partición aleatoria, SKILL.md del proyecto, secciones 1 y 3); (2) la prueba de la celda 38: en la partición aleatoria el **87,0 %** de las filas de prueba tienen un año vecino de su serie en entrenamiento, señal inequívoca de fuga. |
| **Corrección aplicada** | En las celdas 13–16: etiqueta binaria `Y = 1` si `produccion_toneladas(t) < 0,8 × mediana(5 años anteriores)` (celda 13), partición cronológica estricta 2005–2018 / 2019–2020 / 2021–2022 (celda 16), salida de un logit con `BCEWithLogitsLoss` (celdas 22–24) y línea base de persistencia y clase mayoritaria (celda 18). Se comprobó con la prueba automática de no filtración (celda 13 → «OK») y con el assert de no solapamiento de periodos (celda 16). |
| **Reproducción** (la exige el nivel Estratégico) | Celda 38 · versión de la IA: partición **aleatoria** 70/30 con `LogisticRegression` (`class_weight='balanced'`) sobre los mismos datos y la semilla 42 · versión corregida: partición **cronológica** (≤2018 vs ≥2021) con el mismo clasificador y la misma semilla · medición común: accuracy y recall de la clase caída en prueba |
| **Resultado de la medición** | Partición aleatoria: accuracy **0,729** · recall caída **0,731** · partición cronológica: accuracy **0,662** · recall caída **0,709** · fuga medida: **87,0 %** de filas de prueba con año vecino en entrenamiento |
| **Lección** | En series temporales no se puede confiar en una partición aleatoria ni en las métricas que produce: la fuga de años vecinos crea una ventaja ilusoria (aquí ≈ +0,067 de accuracy y +0,022 de recall). La IA propone recetas estándar sin considerar la pregunta de negocio ni la estructura de los datos; cada sugerencia debe auditarse contra la teoría del curso (no filtración, partición cronológica, línea base) antes de aceptarla. |

*(La reproducción de la celda 38 mantiene el mismo fenómeno para el MLP de la v1: la celda usa regresión logística con la misma semilla sobre los mismos datos para aislar el efecto de la partición; la sección 14 del cuaderno registra que el fallo de la v1 se corrige en las secciones 4, 5 y 8.)*
