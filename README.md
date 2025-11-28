# Proyecto de Machine Learning

## Segmentación de Clientes mediante Clustering (K-Means)

Este proyecto desarrolla un flujo completo de _Machine Learning no supervisado_ para segmentar clientes de Home Credit usando **K-Means**.  
El trabajo se divide en **tres fases**: comprensión, preparación, modelamiento y evaluación.

Para ejecutar correctamente el código, es necesario crear una carpeta llamada datos_examen y colocar en ella los archivos .parquet incluidos en el archivo
datos_examen.zip. Esta carpeta debe ubicarse en el mismo directorio donde se encuentra el Notebook.

---

# 1. Comprensión de los datos

En esta etapa analizamos la estructura del dataset original para entender su composición y calidad.

### Exploración inicial

- Recuento de registros y variables: **307.511 filas** y **122 columnas**.
- Identificación de tipos de variables:
  - **Numéricas continuas:** ingresos, monto de crédito, anuitidad, antigüedad, etc.
  - **Categóricas:** género, tipo de contrato, educación, tipo de vivienda, ocupación, etc.
  - **Fechas expresadas en días negativos**.
- Análisis descriptivo de cada columna:
  - Estadísticos básicos (min, max, media, percentiles).
  - Detección de outliers fuertes (AMT_INCOME_TOTAL, AMT_CREDIT).
  - Revisión de distribuciones y cardinalidades.
- Evaluación de valores faltantes:
  - Algunas variables presentan más de **50–70% de nulos**.
- Revisión preliminar de posibles redundancias:
  - `REGION_RATING_CLIENT` vs `REGION_RATING_CLIENT_W_CITY`
  - `CNT_CHILDREN` vs `CNT_FAM_MEMBERS`
  - Variables altamente correlacionadas de crédito.

### Ingeniería de características

Se crean nuevas columnas:

- `AGE_YEARS`
- `EMPLOYED_YEARS`

---

# 2. Preparación de los datos

Esta fue la fase más extensa. Aplicamos un pipeline basado en limpieza, transformación y reducción.

### Eliminación de columnas irrelevantes

Se descartaron columnas según criterios:

- **ID o sin relación con comportamiento:** `SK_ID_CURR`
- **Target (para evitar data leakage en clustering):** `TARGET`
- **Alto procentaje de nulos**
- **Varianza extremadamente baja**
- **Cardinalidad excesiva**
- **Correlacion irrelevante**
- **Descarte lógico**

_Resultado:_ reducción de 124 → **25 columnas útiles**.

---

### Imputación de valores faltantes

**Numéricas:**

- Se imputó con **mediana** por la asimetría de las columnas.

**Categóricas:**

- Se imputó con la **moda**.
- En `OCCUPATION_TYPE = None`, se mantuvo como categoría válida (desempleado/no informado).

---

### Encoding de variables categóricas

Dos estrategias:

1. **Ordinal Encoding** para educación  
   (`Lower secondary` -> `Academic degree`)

2. **One-Hot Encoding** para variables nominales
   - Contract type
   - Gender
   - Housing type
   - Income type
   - Occupation
   - Organization type  
     Etc.

_Resultado:_ dataset final de 40 columnas.

---

### Escalado de variables

Se compararon **StandardScaler vs MinMaxScaler**:

- MinMax mostró:
  - Mejor comportamiento en variables fuertemente asimétricas.
  - Menor sensibilidad a outliers.
  - **Menor inercia de KMeans**, generando clusters más compactos.

**Elección final: MinMaxScaler**

---

# 3. Modelamiento

### Determinación del número óptimo de clusters

Se utilizaron dos métodos:

- **Elbow Method** caída significativa entre 700000 y 650000 hasta K=3.
- **Silhouette Score:** mejor puntaje (0.21) para K=3.

**Se eligió K = 3 clusters.**

---

### Entrenamiento de K-Means

- Se entreno con km = KMeans(n_clusters=3, random_state=42)

---

# 4. Evaluación

Se hizo una evaluación interna usando:

- **Silhouette Score** El valor Silhouette Score k=3: 0.1823 obtenido indica que los grupos descubiertos por el modelo existen, pero 
  presentan un grado considerable de traslape.

- **Davies–Bouldin Index** El valor DBI = 2.2430 obtenido confirma que los clusters son aceptables pero no estan fuertemente diferenciados.


# 5. Conclusion

El modelo de clustering logra capturar patrones generales y produce una segmentación para exploración inicial, pero no presenta separación robusta entre grupos.
Los resultados no son determinantes.
