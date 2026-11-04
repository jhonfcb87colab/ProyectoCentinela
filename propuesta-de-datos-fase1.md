# Propuesta de Datos · Proyecto Centinela · Fase 1

Una página de texto (secciones 0 a 8) más tres anexos que no cuentan en la extensión: A. prueba de acoplamiento; B. evidencias del test de procedencia; C. checklist de protección de datos (uno por fuente).

## 0. Encabezado

| Campo | Detalle |
| --- | --- |
| Título del proyecto | Centinela · Alerta temprana de caída de producción frutícola en el Valle del Cauca |
| Integrantes | John Fernando Cruz Becerra (entrega individual) |
| Fecha | 2026-11-04 |
| Versión | v2 (reescritura metodológica tras la retroalimentación del docente) |

## 1. Problema, cliente y objetivo binario

- **Evento de riesgo y cliente:** caída anual de la producción de frutales (por serie `id_municipio` – `cultivo` – `ciclo`) para la Secretaría de Agricultura y Desarrollo Rural del Valle del Cauca y las asociaciones de productores.
- **Impacto:** una caída del 20 % o más respecto del nivel habitual compromete el abastecimiento regional, el cupo de transporte refrigerado y los programas de apoyo; alertar con anticipación permite movilizar asistencia técnica, seguros y logística.
- **Objetivo binario:** `Y = 1` si `produccion_toneladas(t) < 0,80 × mediana(producciones t−1 … t−5)` de la misma serie; `Y = 0` en caso contrario. El fenómeno es continuo y el umbral del 80 % lo vuelve binario: es el punto en que la caída deja de ser variabilidad normal y pasa a ser un evento que exige decisión pública (justificado por el costo que una caída no alertada implica para la Secretaría).

## 2. Las dos modalidades del sistema completo

- **Rama de imagen:** imágenes ópticas Sentinel-2 (resolución 10 m) del Copernicus Data Space Ecosystem; cada imagen representa la reflectancia del terreno en el año de producción. Aporta el NDVI agregado **solo sobre píxeles con máscara de cultivo** (no sobre el polígono municipal completo), es decir, el estado fenológico y el vigor del dosel frutal durante el ciclo.
- **Rama temporal:** producción anual en toneladas por municipio–cultivo–ciclo (portal `datos.gov.co`, dataset `8xap-gecf`, cobertura 2000–2022, frecuencia anual) y, en la Fase 2, variables climáticas ERA5-Land (~9 km, agregadas a anual) del Copernicus Climate Data Store.
- **Pregunta de control, «¿La imagen contiene algo que un número no contenga?»:** Sí. El NDVI con máscara de cultivo contiene el estado del dosel frutal del propio año (vigor, fenología), información que la producción anual agregada no contiene; por eso el sistema completo tiene dos modalidades (el notebook las acopla por la clave `(id_municipio, anio)`).

## 3. Fuentes (con enlaces y licencia)

| Modalidad | Fuente (productor primario) | Enlace o DOI | Licencia y versión | Dónde se verificó la licencia | ¿Datos de personas? |
| --- | --- | --- | --- | --- | --- |
| Imagen | Sentinel-2 MSI (NDVI) · Copernicus Data Space Ecosystem (ESA) | https://dataspace.copernicus.eu/ | Datos Copernicus Sentinel: uso libre y gratuito, con atribución («Copernicus Sentinel data, [año]») | Portal de acceso Copernicus (términos y condiciones del Data Space Ecosystem) | No (imágenes de satélite; no muestran personas) |
| Temporal | Producción de frutales en el Valle del Cauca · Gobernación del Valle del Cauca (portal `datos.gov.co`, dataset `8xap-gecf`) | https://www.datos.gov.co/api/v3/views/8xap-gecf/query.json | CC BY-SA 4.0 (Creative Commons Atribución–CompartirIgual) | Metadatos del dataset en la API de datos.gov.co (campo `license`: `CC_40_BY_SA`), consultados en el cuaderno, celda 5 | No (registros agregados por municipio, cultivo y ciclo; sin personas) |

En la Fase 2 se suma ERA5-Land (clima) del Copernicus Climate Data Store, de acceso abierto, con atribución al C3S.

## 4. Por qué NO es un reto resuelto

1. **El dato es crudo:** se descarga en el cuaderno directamente de la API oficial de `datos.gov.co` (celda 5) con registro de fecha de descarga, tamaño (3 106 210 bytes) y huella SHA-256 **antes** de cualquier transformación.
2. **No proviene de Kaggle ni de una réplica suya:** la celda 6 consulta la API de búsqueda de Kaggle y devuelve **0 resultados** para «produccion frutales valle del cauca»; el origen declarado es la API Socrata del portal oficial.
3. **No hay solución pública que baste con copiar:** la etiqueta binaria (mediana de 5 años, sin filtración), la partición cronológica, la línea base de persistencia y la evaluación frente al MLP se construyen en el cuaderno sobre el dato crudo, con prueba automática de no filtración y reproducción del fallo de la v1.

Las evidencias completas del test de procedencia se documentan en el anexo B.

## 5. Plan de la Fase 1, clave de alineación y prueba de acoplamiento

- **Modalidad que se modela en la Fase 1 (aplanada a tabla):** la rama temporal tabular — producción anual de frutales 2000–2022, aplanada a **una fila por `(id_municipio, cultivo, ciclo, anio)`**: 9 934 registros, de los cuales 6 764 quedan etiquetables (faltan 5 años previos en el resto). Predictores: categóricos (`municipio`, `subregion`, `cultivo`, `ciclo`) y numéricos (`anio`, `lag1`, `lag2`, `lag3`, `mediana_5_prev`, `ratio_lag1`, `tendencia_3`) → vector de 85 características.
- **Paso a la Fase 2:** arquitectura nativa — CNN para la rama de imagen NDVI y RNN/LSTM/GRU para la serie climática, con incorporación de la segunda modalidad por la clave agregada `(id_municipio, anio)`; el MLP de la Fase 1 queda como línea base del sistema multimodal.
- **Clave de alineación (se marca una):**
  - [ ] Agregada, p. ej., (región, mes), con su salvedad de fuga de datos y de rasgo casi constante: no elegida; la llave concreta del proyecto es por muestra (*ver casilla marcada*).
  - [x] Por muestra, p. ej., (finca, semana): llave concreta (entidad + ventana) y fuente fechada y georreferenciada que la habilita: **llave `(id_municipio, anio)`** (entidad = municipio con código DANE, ventana = año), habilitada por la fuente anual de producción por municipio, fechada (2000–2022) y georreferenciada (42 municipios del Valle del Cauca). Salvedad declarada: varios cultivos del mismo municipio–año **heredan el mismo vector NDVI/climático** (el cuaderno lo declara y evalúa la Fase 2 como señal complementaria, no como predictor individual).
- **Prueba de acoplamiento (obligatoria):** diez filas reales en el anexo A.
- **Si la modalidad aplanada es una serie temporal:** sí lo es. **Partición cronológica estricta:** entrenamiento ≤ 2018, validación 2019–2020, prueba ≥ 2021; los rezagos (`lag1`–`lag3`), la mediana y las transformaciones se calculan sobre la serie completa 2000–2022 con `shift(1)` antes de la ventana móvil (sin mirar el futuro), verificado con una prueba automática sobre 200 muestras (celda 13).

## 6. Línea base comprometida

- **Clase mayoritaria** (alertar siempre «sin caída») en el conjunto de prueba 2021–2022 (n = 764): **Accuracy 0,649** y **recall de la clase de riesgo 0,000** (el *recall* vale 0 porque la clase mayoritaria es «sin riesgo»).
- **Escalar crudo** (la modalidad aplanada resume un escalar, toneladas): línea base de **persistencia** — alertar caída en `t` si `t−1` fue caída (`lag1 < 0,8 × mediana_5_prev`), la regla que la Secretaría podría aplicar sin modelo. En prueba: **Accuracy 0,792 · recall caída 0,612 · precisión 0,749 · F1 0,674** (el MLP no la supera en exactitud ni F1; solo la supera en recall, ver notebook, celda 40).
- La partición cronológica, la semilla **42** y estas cifras se conservan: son el punto de comparación de la Fase 2.

## 7. Riesgos éticos y protección de datos

| Riesgo | Daño para el cliente | Mitigación |
| --- | --- | --- |
| Sesgo de muestreo o representatividad | Municipios o cultivos con pocas observaciones reciben probabilidades menos fiables; el patrón de caídas cambia con los años (prevalencia 28,5 % train / 29,0 % val / 35,1 % test). | Reportar métricas por subregión y cultivo; no decidir sobre series con menos de 5 años; reentrenamiento anual y monitoreo del recall por año. |
| Falsa alarma | Sobrecosto de logística y visitas técnicas innecesarias en la red de apoyo. | Segunda verificación con datos climáticos y de campo en la Fase 2; el modelo es soporte a la decisión, no un dictamen. |
| Alarma omitida | Sin apoyo oportuno: pérdida de cosecha, sobreprecio de abastecimiento, productores sin asistencia. Es el error más costoso. | Umbral orientado a *recall* elegido en validación con costo explícito FN:FP = 3:1 (umbral operativo 0,55) y revisión humana de las alertas de alta probabilidad. |

El checklist de protección de datos va en el anexo C, uno por fuente.

## 8. Cita APA 7 de cada fuente

- Gobernación del Valle del Cauca. (2024). *Producción de frutales en el Valle del Cauca* [Conjunto de datos]. datos.gov.co. Recuperado el 4 de noviembre de 2026, de https://www.datos.gov.co/d/8xap-gecf
- European Space Agency. (2025). *Sentinel-2* [Misión de observación de la Tierra]. Copernicus Data Space Ecosystem. https://dataspace.copernicus.eu/
- Muñoz Sabater, J. (2019). *ERA5-Land monthly averaged data from 1950 to present* [Conjunto de datos]. Copernicus Climate Change Service (C3S) Climate Data Store. https://doi.org/10.24381/cds.68d2bb30
- Guío Español, A., Tamayo Uribe, E. y Gómez Ayerbe, P. (2021). *Marco ético para la inteligencia artificial en Colombia*. Presidencia de la República de Colombia.

---

## Anexo A · Prueba de acoplamiento

Clave: (id_municipio, anio) — entidad = municipio (código DANE), ventana = año

| N.º | Clave | Dato de la rama de imagen (archivo o rasgo) | Dato de la serie (variable y valor) |
| --- | --- | --- | --- |
| 1 | Sevilla · 2021 | NDVI medio anual (Sentinel-2, máscara de cultivo): 0,62* | produccion_toneladas = 30 101 (plátano, anual) |
| 2 | Sevilla · 2022 | NDVI medio anual: 0,66* | produccion_toneladas = 77 832 (plátano, anual) |
| 3 | La Unión · 2021 | NDVI medio anual: 0,58* | produccion_toneladas = 3 658 (melón) |
| 4 | La Unión · 2022 | NDVI medio anual: 0,55* | produccion_toneladas = 2 534 (melón) |
| 5 | Toro · 2021 | NDVI medio anual: 0,57* | produccion_toneladas = 1 146 (melón) |
| 6 | Toro · 2022 | NDVI medio anual: 0,59* | produccion_toneladas = 1 587 (melón) |
| 7 | Roldanillo · 2021 | NDVI medio anual: 0,56* | produccion_toneladas = 1 080 (melón) |
| 8 | Roldanillo · 2022 | NDVI medio anual: 0,54* | produccion_toneladas = 1 000 (melón) |
| 9 | Yumbo · 2021 | NDVI medio anual: 0,64* | produccion_toneladas = 0 (banano) |
| 10 | Yumbo · 2022 | NDVI medio anual: 0,63* | produccion_toneladas = 0 (banano) |

*\* NDVI, temperatura y precipitación ilustrativos del esquema de acoplamiento; se sustituyen por los promedios reales de Sentinel-2 y ERA5-Land al construir la Fase 2 (declarado en `04Propuesta_Proyecto_Centinela.txt`). La producción es real: la de Sevilla coincide con la serie del cuaderno (celda 10, plátano · Sevilla · anual).*

Código o consulta que produjo las filas: sección 3 del cuaderno (celdas 9–10), que construye `df` con la clave `(id_municipio, cultivo, ciclo, anio)` y `produccion_toneladas` desde la API oficial; las filas del acoplamiento multimodal se toman de la tabla de 10 ejemplos de `04Propuesta_Proyecto_Centinela.txt` (sección 3), que es la consulta de alineación por `(id_municipio, anio)` declarada en el notebook (celdas 1 y 39).

## Anexo B · Evidencias del test de procedencia

1. **Cadena de procedencia** (un archivo por fila):

| Archivo (nombre exacto) | URL o DOI del productor primario | Fecha de descarga | Huella SHA-256 |
| --- | --- | --- | --- |
| `8xap-gecf` (respuesta JSON cruda de la API, 3 106 210 bytes) | https://www.datos.gov.co/api/v3/views/8xap-gecf/query.json | 2026-11-04 | `5f2ec835628e8dc28768982a9756775e8962278b3f92ad472a46b61089ea562c` |

2. **Descarga trazable:** celda 5 del cuaderno: `requests.get(ENDPOINT)` a la API oficial de datos.gov.co (Socrata), con fecha UTC, tamaño del JSON crudo y SHA-256 calculado antes de transformar; la búsqueda de «kaggle» en el cuaderno (celda 6) devuelve **0 resultados** y la conclusión escrita es «la base se obtiene de la API oficial del portal; no se usó Kaggle ni espejos del reto».
3. **Búsqueda declarada de espejos** (2026-11-04): consulta a la API de Kaggle `https://www.kaggle.com/api/v1/datasets/list?search=produccion%20frutales%20valle%20del%20cauca` → **sin coincidencias (0 resultados)**.
4. **Huella de crudeza** (antes de limpiar): filas × columnas — 9 934 × 7 tras la limpieza (se eliminaron 39 duplicados exactos y se promediaron 58 duplicados de la clave `(id_municipio, cultivo, ciclo, anio)`) · faltantes — 0 % después del `dropna` de `produccion_toneladas`/`anio` · balance de la etiqueta — 29,3 % de caída (Y = 1) en las 6 764 filas etiquetables.
5. **Contraste con el espejo** (si existe): no existe espejo (0 resultados en Kaggle); no aplica contraste.

## Anexo C · Checklist de protección de datos personales (uno por fuente)

Fuente: Producción de frutales en el Valle del Cauca (datos.gov.co, dataset `8xap-gecf`)

Regla de la Fase 1: sin personas identificables. Solo se usan datos públicos, agregados o anonimizados por la fuente oficial. «Ley» = Ley 1581 de 2012; «DUR» = Decreto 1074 de 2015, que compila el Decreto 1377 de 2013.

**Resultado** (se marca uno):

- [x] Sin datos de personas.
- [ ] Datos de personas anonimizados por la fuente.
- [ ] Datos personales o sensibles → no admisibles en la Fase 1; en fases posteriores requieren el visto bueno del docente antes de modelar.

### A. ¿Hay datos personales? (Ley, art. 3 lit. c)

- [x] Se revisaron todas las columnas, imágenes y metadatos en busca de identificadores directos (nombre, documento, correo, teléfono, dirección, número de cuenta, contrato, NIU, matrícula, placa, rostro, voz). La base contiene solo producción agregada por municipio, cultivo, ciclo y año.
- [x] Se revisaron los identificadores indirectos, que combinados permiten señalar a alguien: coordenadas exactas, fecha + municipio pequeño, edad + sexo + institución, metadatos EXIF/GPS de fotos. El único identificador es el código DANE oficial del municipio, que identifica un territorio, no una persona.
- [x] Un identificador seudonimizado (p. ej., un NIU o un documento cifrado) se trata como dato personal: sigue siendo «asociable» a la persona. No hay identificadores seudonimizados en la base.

### B. ¿Hay datos sensibles? (Ley, art. 5)

*(No aplica: la base no contiene datos de salud, origen, orientación, políticos ni biométricos; las unidades son municipios, no personas.)*

- [ ] Si los hay, la base es una de estas dos: autorización explícita del titular (art. 6 lit. a; DUR 2.2.2.25.2.3) o finalidad científica o estadística con supresión de identidad (art. 6 lit. e). En este curso, la regla práctica es usar solo conjuntos anonimizados por la fuente oficial.
- [ ] Imágenes médicas y conteos de salud (p. ej., SIVIGILA): se declaran como datos de salud y no se cruzan con fuentes que permitan reidentificar.

### C. ¿Hay niños, niñas o adolescentes? (Ley, art. 7; DUR 2.2.2.25.2.9)

*(No aplica: no hay titulares menores de edad ni registros escolares, pediátricos o de matrícula.)*

- [ ] Si los titulares pueden ser menores (p. ej., pruebas de Estado escolares, matrícula, registros pediátricos), solo se usan datos públicos o anonimizados y agregados de fuente oficial. No se recolectan datos propios de menores en este proyecto.

### D. Base legal o autorización para datos de terceros

- [x] Fuente pública oficial: se citan sus términos de uso. La fuente es el portal oficial de datos abiertos de Colombia (datos.gov.co) con licencia CC BY-SA 4.0 verificada en los metadatos. Que algo esté «en internet» no lo convierte en dato público (DUR 2.2.2.25.2.2): aquí la publicación es formal del Estado, con licencia explícita.
- [ ] Datos propios o de una organización (finca, cooperativa, empleador): hay autorización previa, expresa e informada del titular o permiso escrito de la entidad (Ley, arts. 3 lit. a y 9; DUR 2.2.2.25.2.4: escrita, oral o por conducta inequívoca, «en ningún caso el silencio»), y se conserva la prueba (DUR 2.2.2.25.2.5). En la Fase 1, además, con las salvaguardas de los datos propios no queda ninguna persona identificable.
- [ ] Datos de un empleador: además, hay permiso para usarlos fuera de la empresa y se respeta su confidencialidad (Ley, art. 4 lit. h).

### E. Finalidad (Ley, art. 4 lit. b)

- [x] Finalidad declarada: «uso académico para entrenar y evaluar un modelo en el curso; no se toman decisiones sobre personas identificadas» (celda 7 del cuaderno: alerta temprana de caída de producción para planeación pública, uso estadístico agregado).

### F. Anonimización y riesgo de reidentificación

- [x] Se eliminaron los identificadores directos y se generalizaron los indirectos (p. ej., coordenadas → vereda o municipio; fecha → semana o mes; edad → rangos). No había identificadores directos: el dato ya es agregado por municipio, cultivo y ciclo.
- [x] Se borraron los metadatos EXIF/GPS de las imágenes. No hay imágenes propias en la Fase 1 (las imágenes Sentinel-2 llegan en la Fase 2 y no contienen personas).
- [x] Prueba de reidentificación respondida por escrito: «¿podría alguien identificar a una persona combinando estas columnas con información pública?» → No: las columnas describen producción de municipios, cultivos y ciclos, no personas; las celdas con conteos pequeños (p. ej., producción 0 en Yumbo) describen territorios, no individuos, y no se publican registros individuales.

### G. Licencia de uso (de cada fuente, verificada en la fuente primaria)

- [x] Nombre y versión de la licencia, con el enlace donde se verificó: **CC BY-SA 4.0**, verificada en los metadatos del dataset en la API de datos.gov.co (campo `license` = `CC_40_BY_SA`, celda 5 del cuaderno).
- [x] Obligaciones anotadas: BY = atribuir e indicar los cambios (atribución a la Gobernación del Valle del Cauca en la cita APA y en `04Propuesta_Proyecto_Centinela.txt`); SA = los derivados redistribuidos van con la misma licencia (aplica a características tabulares derivadas y a redistribuciones del conjunto); NC = no (el cliente es una entidad pública); ND = no.
- [x] Se cumplen los términos del portal (cita obligatoria de la fuente; la descarga se hace por la API pública oficial sin registro; no hay prohibición de reidentificar porque no hay personas).

### H. Transferencia o transmisión internacional (Colab, Drive, GitHub, asistentes de IA)

- [x] Se sabe que estos servicios procesan los datos fuera de Colombia (Ley, art. 26; DUR 2.2.2.25.1.3, nums. 4–5, y 2.2.2.25.5.1). Aunque el país de destino figure en la lista de la SIC (Circular Única, Título V, num. 3.2), se exige responsabilidad demostrada.
- [x] A Colab, Drive y GitHub solo se suben datos sin personas identificables (públicos, anonimizados o agregados). El cuaderno y el repositorio contienen solo producción agregada por municipio.
- [x] A los asistentes de IA no se les pegan registros reales con datos personales: solo el esquema o filas sintéticas. En este proyecto se compartieron con la IA el esquema (columnas y tipos) y filas agregadas sin personas (ver bitácora, sección 3).

### I. Minimización y conservación

- [x] Solo se descargan y conservan las columnas necesarias para el problema (DUR 2.2.2.25.2.1): se eliminan las columnas internas de Socrata (prefijo `:`) y solo se usan las 7 columnas del modelo.
- [x] Se declara dónde se guardan los datos crudos y que se borran al terminar el curso o la Fase 3 (DUR 2.2.2.25.2.8), con medidas de seguridad razonables (Ley, art. 4 lit. g): el JSON crudo vive en memoria del cuaderno y su integridad se verifica con el SHA-256 registrado; el repositorio conserva el script de descarga, no el dato crudo; se eliminarán las copias locales al finalizar la Fase 3.

### J. Publicación del repositorio (Ley, art. 4 lit. f)

- [x] El repositorio es privado y compartido con el docente, o es público y no contiene datos personales y la licencia de los datos permite redistribuirlos. Repositorio con control de acceso (celda 7 del cuaderno): https://github.com/jhonfcb87colab/ProyectoCentinela.
- [x] Si los datos no se pueden redistribuir, el repositorio trae el script de descarga desde la fuente primaria; la carpeta de datos va en `.gitignore`. El cuaderno descarga los datos de la API oficial al ejecutarse; los datos crudos no se incluyen en el repositorio.
- [x] Las salidas del cuaderno (`df.head()`, ejemplos de imágenes, capturas) no muestran registros con datos personales. Las salidas publicadas son `df.head()` con producción por municipio (Cali, 76001) y las tablas de métricas agregadas.

**Declaración.** Lo consignado en este checklist es verdadero y se revisó para cada fuente de la Propuesta.

| Integrante | Firma | Fecha (AAAA-MM-DD) |
| --- | --- | --- |
| John Fernando Cruz Becerra | _________JHONFCB_______________ | 2026-11-04 |
