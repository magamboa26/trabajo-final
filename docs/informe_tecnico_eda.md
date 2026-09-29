# Informe técnico del análisis exploratorio de datos

## Cartera de crédito de la banca privada del Ecuador

**Periodo analizado:** enero-agosto de 2026  
**Fuente:** Superintendencia de Bancos del Ecuador, Portal Estadístico, sección *Cartera y Depósitos*.  
**Unidad de análisis:** cartera de crédito por segmento, entidad, provincia, cantón y mes.

## Objetivo

El análisis responde a la pregunta: **¿qué segmentos, regiones y entidades concentran la morosidad de la banca privada, y qué diferencias se observan en el periodo?** Su propósito es orientar la exploración del riesgo y señalar dónde conviene profundizar la revisión. No estima provisiones ni demuestra causas del deterioro.

La cartera improductiva se definió como la suma de cartera que no devenga intereses y cartera vencida. La morosidad se calculó como:

> **Morosidad (%) = cartera improductiva / saldo total × 100**

Al comparar grupos, la tasa se calculó de forma ponderada: suma de cartera improductiva dividida para la suma de saldos. No se promediaron directamente los porcentajes de las filas.

## Protocolo EDA

### 1. Pregunta de análisis

Se planteó identificar dónde se concentra el riesgo de crédito para ayudar a priorizar el monitoreo de segmentos, regiones y entidades. Como hipótesis exploratorias se propuso que microcrédito podría presentar más morosidad que productivo, que las tasas podrían variar entre regiones y que una parte importante de la cartera improductiva podría concentrarse en pocas entidades.

La hipótesis contrastada estadísticamente fue más acotada: **en agosto de 2026, la morosidad por entidad es mayor en microcrédito que en productivo**. El alcance se limitó a entidades con datos disponibles para ambos segmentos.

### 2. Lectura y comprensión de los datos

Se leyeron las hojas BASE de los cuatro archivos Excel de consumo, microcrédito, productivo y vivienda. La lectura inicial produjo **13.778 filas** correspondientes a ocho cortes mensuales, desde el 31 de enero hasta el 31 de agosto de 2026. Se revisaron dimensiones, tipos de datos, fechas, cobertura de segmentos y cantidad de entidades, provincias y cantones.

Las variables originales incluyen segmento, fecha, entidad, provincia, cantón y los saldos por vencer, que no devengan intereses, vencidos y totales. Se comprobaron dos controles importantes:

- La suma de los tres componentes de cartera coincide con el saldo total, con una diferencia máxima de aproximadamente USD 0,000001 por precisión numérica.
- Se encontraron 13 filas repetidas en la llave segmento-fecha-entidad-provincia-cantón. Se consolidaron sumando los saldos, con lo que quedaron **13.765 observaciones**.

Después se derivaron el mes, la región asignada a partir de la provincia, la cartera improductiva y la tasa de morosidad por fila. El dataset limpio se guardó en `data/processed/cartera_banca_privada_2026.csv`.

### 3. Análisis univariado

Se describió cada variable por separado para conocer la cobertura y la distribución. Se contaron registros por segmento y mes; se calcularon media, desviación, mínimo, cuartiles, percentil 95, máximo y asimetría para saldo, cartera improductiva y morosidad. Se visualizaron histogramas, usando escala logarítmica para el saldo por su amplio rango, y se compararon saldos y tasas agregadas por segmento y entidad en agosto.

El saldo presenta fuerte asimetría: la mediana por fila es aproximadamente **USD 2,30 millones**, mientras que la media es **USD 31,04 millones**. La asimetría del saldo es 13,95. Por ello, la media por sí sola no representa bien una observación típica. En agosto, la morosidad ponderada fue **6,38% en microcrédito**, **4,96% en consumo**, **2,82% en inmobiliario**, **1,34% en vivienda de interés público** y **1,07% en productivo**. Aunque consumo tuvo menor tasa que microcrédito, concentró **64,3%** de la cartera improductiva, aproximadamente USD 1.100 millones, por el tamaño de su cartera.

### 4. Análisis bivariado

Se analizaron relaciones entre pares de variables para descubrir diferencias y patrones:

- **Mes y segmento:** se graficó la morosidad ponderada mensual por tipo de crédito, agregando primero los saldos de cada grupo.
- **Segmento y región:** se usó un mapa de calor con la morosidad ponderada de agosto; las celdas sin cartera no se interpretan como tasa cero.
- **Saldo y morosidad:** se elaboró un diagrama de dispersión por entidad-cantón con eje de saldo logarítmico y se calculó la correlación de Spearman. El coeficiente observado fue **0,01**, una asociación monotónica prácticamente nula en este corte. La inspección también mostró que las tasas de las filas pequeñas varían más.
- **Segmento y morosidad:** se compararon las distribuciones con diagramas de caja antes del tratamiento de atípicos.

Los gráficos describen asociaciones; no permiten concluir que una característica cause otra. Para comparar microcrédito y productivo se hizo además una prueba inferencial específica, descrita en la sección de hallazgos.

### 5. Revisión de faltantes y tratamiento de atípicos

Se revisaron valores faltantes y montos imposibles. No se encontraron valores nulos en las variables revisadas, saldos iguales a cero ni montos negativos; por tanto, no se imputaron datos.

Se aplicó la regla del rango intercuartílico (IQR) como diagnóstico univariado. Se marcaron **1.693 filas (12,3%)** como atípicas en saldo y **1.560 (11,3%)** en morosidad. Esta regla identifica valores extremos estadísticos, pero no implica por sí sola que sean errores: un saldo alto puede corresponder a una cartera real de gran tamaño.

Para evitar que tasas de filas diminutas distorsionen comparaciones locales, se identificaron **2.173 filas (15,8%)** con saldo menor a USD 100.000. Esas filas representan solo **0,014% del saldo total** y **0,063% de la cartera improductiva**. De 833 filas con morosidad de al menos 99,9%, 826 están bajo ese umbral. Por esta razón:

- Se conservaron todos los registros en las tasas agregadas de sistema, segmento, región, mes y entidad, donde las tasas se ponderan por saldo.
- Se aplicó el umbral de USD 100.000 a los rankings locales cantón-entidad y al conjunto del modelo, como criterio analítico documentado.
- No se eliminaron ni winsorizaron los importes extremos de la fuente.

### 6. Segmentación

Se separaron los resultados por segmento de crédito, mes, región, provincia, entidad y cantón. Para cada grupo se contrastó la tasa con los dólares improductivos y el tamaño de cartera, evitando confundir una tasa alta sobre una cartera pequeña con una exposición absoluta alta.

En agosto, **Oriente registró la tasa regional más alta (5,65%)**, sobre un saldo aproximado de USD 598 millones. **Sierra** registró 3,43% y concentró la mayor cartera improductiva en dólares, cerca de **USD 1.058 millones**. Cinco entidades reunieron **75,5%** de la cartera improductiva del sistema. En la comparación de enero con agosto, la morosidad de microcrédito bajó 0,37 puntos porcentuales (6,76% a 6,38%); productivo se mantuvo cerca de 1,1%. El crecimiento de vivienda de interés público se interpreta con cautela porque partió de una base de saldo pequeña.

### 7. Hallazgos e interpretación

La morosidad ponderada total de la banca privada fue **3,10% en agosto**, frente a 3,17% en enero. Microcrédito tuvo la mayor tasa por segmento (6,38%); productivo tuvo la menor (1,07%). Consumo representó la mayor exposición en dólares improductivos. Oriente destacó por tasa regional y Sierra por monto absoluto. Las cinco entidades de mayor exposición concentraron tres cuartas partes de la cartera improductiva.

#### Prueba estadística y verificación de supuestos

La comparación se realizó con **21 entidades** que tenían datos de microcrédito y productivo en agosto. Se calculó para cada entidad la diferencia entre ambas tasas ponderadas, en puntos porcentuales. Antes de elegir la prueba se evaluaron los supuestos de la diferencia pareada:

1. **Normalidad:** Shapiro-Wilk produjo W = 0,855 y p = 0,0051. Con α = 0,05 se rechazó la hipótesis nula de normalidad. Esto no es la probabilidad de que la hipótesis nula sea verdadera; indica que los datos observados son incompatibles con normalidad bajo ese modelo.
2. **Valores atípicos:** la regla IQR detectó dos diferencias fuera de los límites de −10,16 y 18,64 puntos porcentuales.
3. **Independencia:** las observaciones corresponden a entidades diferentes. La independencia entre entidades se considera un supuesto del diseño; no puede comprobarse con Shapiro-Wilk y puede verse afectada por factores macroeconómicos compartidos.

Como no se cumplieron los criterios establecidos para usar la prueba paramétrica, se eligió la **prueba binomial de los signos unilateral**, que no requiere normalidad ni simetría de las diferencias. Bajo la hipótesis nula, la probabilidad de que una diferencia sea positiva es 0,5; la alternativa plantea que sea mayor que 0,5. Se observaron **17 diferencias positivas de 21**. La diferencia mediana fue **+3,81 puntos porcentuales** y el p-valor fue **0,0036**. Al nivel de 5%, se rechaza la hipótesis nula: hay evidencia de que la tasa por entidad es mayor en microcrédito que en productivo en este corte. El resultado no prueba causalidad y se refiere a las entidades observadas, no a todos los meses o a otros tipos de instituciones financieras.

#### Modelo de scikit-learn

Se entrenó un `RandomForestRegressor` para estimar la morosidad por fila. La división fue temporal: **10.135 observaciones de enero a julio** para entrenamiento y **1.457 de agosto** para prueba. Se incluyeron segmento, entidad, región, provincia, mes y logaritmo del saldo. Las variables categóricas se codificaron con `OneHotEncoder`; categorías nuevas se toleran con `handle_unknown="ignore"`. Se fijó `random_state=42` y se limitó la complejidad con `min_samples_leaf=5`.

| Modelo | MAE (p.p.) | RMSE (p.p.) | R² |
|---|---:|---:|---:|
| Random Forest | 2,20 | 4,55 | 0,74 |
| Referencia: mediana del entrenamiento | 4,79 | 9,28 | −0,07 |

En agosto, el Random Forest redujo el error absoluto medio frente a la referencia. El RMSE mayor que el MAE refleja errores grandes en algunos registros; R² = 0,74 indica ajuste predictivo en este conjunto de prueba, no una garantía de desempeño en otros periodos. La unidad de evaluación es la fila y las métricas no deben confundirse con una tasa de error del sistema ponderada por dólares.

## Limitaciones y uso responsable

- El periodo contiene solo ocho meses; no alcanza para separar estacionalidad de tendencia ni para validar estabilidad anual.
- Los datos cubren únicamente banca privada ecuatoriana; no representan cooperativas ni banca pública.
- No hay información de provisiones ni recuperaciones; por ello no se estima cobertura ni pérdida esperada.
- La definición de morosidad utilizada es operativa y puede diferir de indicadores oficiales.
- La prueba se refiere a un solo mes y depende de la independencia aproximada de las entidades; los shocks comunes pueden reducirla.
- El modelo se evalúa en un único mes futuro y comparte entidades/territorios con el entrenamiento. El resultado es exploratorio y no acredita generalización a bancos nuevos ni a periodos futuros.
- Las asociaciones y predicciones no establecen causalidad. No deben usarse de forma aislada para decidir provisiones, políticas de crédito o ampliaciones de cartera.

## Reproducibilidad

Desde la carpeta `proyecto/`, instalar las dependencias declaradas y abrir el notebook:

```bash
pip install -r requirements.txt
jupyter notebook notebooks/EDA_cartera_banca_privada.ipynb
```

El notebook localiza la raíz del proyecto a partir de la ruta de ejecución, lee los archivos de `data/raw/`, fija la semilla del modelo en 42 y escribe el CSV limpio en `data/processed/`. Para replicar los resultados, ejecutar las celdas en orden desde un kernel limpio.