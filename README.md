# Análisis de la severidad de la pérdida (LGD) y su asociación con las características de los créditos en default de Happen Bank (LendingClub, 2007–2018)

 **Equipo:** Daira Julieth Campos V. ([@Dairacampos](https://github.com/Dairacampos)), María Camila Cárdenas C. ([@mariacamilacc1](https://github.com/mariacamilacc1)) y Alejandro José Medina C. ([@AlejandroMedinaCardenas](https://github.com/AlejandroMedinaCardenas)).


**Cursos:** Estadística Bayesiana y Pensamiento Crítico en Ciencia de Datos 2, Universidad Externado de Colombia, 2026-2.


## Abrir los cuadernos en Google Colab

| Cuaderno | Abrir |
|---|---|
| Cuaderno principal | [![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mariacamilacc1/LGD-Happen-Bank/blob/main/Proyecto_LGD_NotebookFinal.ipynb) |
| Ventana de maduración | [![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mariacamilacc1/LGD-Happen-Bank/blob/main/Proyecto_LGD_VentanaMaduracion.ipynb) |
| Exploratorio: recuperación con regresión bayesiana | [![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mariacamilacc1/LGD-Happen-Bank/blob/main/Exploratorio_Reformulacion_RecuperacionBayesiana.ipynb) |
| Exploratorio: recuperación con LightGBM | [![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/mariacamilacc1/LGD-Happen-Bank/blob/main/Exploratorio_Recuperacion_LightGBM.ipynb) |

Al abrir, ir a **Entorno de ejecución → Ejecutar todas**.

## Sobre el proyecto.
El proyecto analiza cuánto pierde Happen Bank (antes LendingClub) cuando un crédito cae en default, medido como la proporción del monto prestado que no se recupera (LGD). Con una regresión bayesiana estima cómo cambia la pérdida según la calificación de riesgo, el plazo, el propósito, el monto, el ingreso y la relación deuda/ingreso del crédito, con intervalos que expresan la incertidumbre de cada estimación.

La decisión que busca mejorar es la **priorización de la cobranza**: si la unidad de cobranza solo puede atender el 20 % de los créditos castigados, a cuáles dedicarles el esfuerzo primero.

**Pregunta:** ¿Cómo puede el banco mejorar la priorización de los créditos en default estimando la severidad de sus pérdidas?

## Datos

- **Fuente:** *All Lending Club loan data* (George, 2019), en Kaggle, con licencia CC0: Public Domain. La copia exacta que usa el proyecto, descargada el 26 de octubre de 2026, está publicada en el Release [`v1.0-avance3`](https://github.com/mariacamilacc1/LGD-Happen-Bank/releases/tag/v1.0-avance3), con su huella SHA-256 en [Versionado de la Base de Datos](Versionado_Base_Datos.md).
- **Población:** 269.320 créditos castigados originados entre 2007 y 2018.
- **Base del análisis:** 196.403 créditos cuya recuperación ya terminó (ventana de maduración de 15 meses desde el default).
- **Partición temporal:** entrenamiento con 2007–2014 (75.175 créditos) y prueba con 2015 (64.846 créditos); 2016–2017 no se usan por ser cohortes incompletas.

El detalle de las variables, su calidad y su traducción está en la [Ficha Técnica de los Datos](Ficha_Tecnica_Datos.md).

## Metodología

1. **Variable objetivo:** LGD = ((monto − capital pagado) − (recuperación − comisión)) / monto, acotada entre 0 y 1.
2. **Censura del target:** ventana de maduración de 15 meses desde el default, elegida sobre la curva de recuperación (ver el anexo de la [Bitácora de Decisiones Analíticas](Bitacora_Decisiones_Analiticas.md)).
3. **Selección de variables:** se excluye la tasa de interés por multicolinealidad con la calificación (VIF de 9,7) y la utilización del crédito por su relación casi nula con la pérdida.
4. **Pipeline:** la preparación de las variables (centrado, logaritmo del ingreso, agrupación de propósitos pequeños y dummies creadas a mano) se aprende solo con los datos de entrenamiento.
5. **Modelo:** regresión lineal bayesiana con priors semiconjugadas (Normal para los coeficientes y Gamma-Inversa para la varianza), estimada con muestreo de Gibbs en 4 cadenas de 6.000 iteraciones, descartando las 1.000 primeras de cada una.
6. **Validación:** validación cruzada temporal con ventana expansiva en tres pliegues (2012, 2013 y 2014) y una sola evaluación final en la prueba de 2015.

## Resultados

| Criterio de éxito | Línea base | Meta | Resultado | ¿Cumple? |
|---|---|---|---|---|
| RMSE de la LGD | Pérdida promedio de 2007–2014 para todos los créditos (RMSE de 0,2059) | Mejora de al menos 10 % | 0,1855 (mejora de 9,9 %) | No |
| Captura@20 % de la pérdida | Orden por saldo al default (40,6 %) | Al menos 40,9 % | 39,2 % | No |

Con las demás características iguales, los créditos de calificación G pierden 13,2 puntos porcentuales más que los de calificación A, y los créditos a 60 meses pierden 10,9 puntos más que los de 36 meses.

**Dictamen:** el proyecto de priorización de la cobranza de Happen Bank **se reformula** porque el orden del modelo concentra el 39,2 % de la pérdida en el 20 % de créditos que cobranza puede atender, por debajo del 40,6 % que ya logra el orden por saldo y del 40,9 % fijado como criterio, y su RMSE mejora solo 9,9 % frente a la línea base, sin alcanzar el 10 % exigido.

## Contenido del repositorio

| Archivo | Contenido |
|---|---|
| `README.md` | Este documento: generalidades del proyecto |
| `Proyecto_LGD_NotebookFinal.ipynb` | Cuaderno principal: carga, limpieza, LGD, censura, EDA, modelo bayesiano, validación y evaluación |
| `Proyecto_LGD_VentanaMaduracion.ipynb` | Análisis y figuras de la ventana de maduración (anexo de la bitácora) |
| `Proyecto_LGD_VersionadoBase.ipynb` | Crea la copia versionada de la base en Google Drive (se corre una sola vez) |
| `Versionado_Base_Datos.md` | Documentación de la copia versionada: fuente, versión, fecha de descarga y huella SHA-256 |
| `Ficha_Tecnica_Datos.md` | Inventario de datos, calidad, brechas y traducción de las variables |
| `Bitacora_Decisiones_Analiticas.md` | Percances y decisiones del proyecto, con el anexo de la ventana de maduración |
| `Analisis_Exploratorio.md` | Alternativas probadas después del resultado, que no cambian el dictamen |
| `Exploratorio_Reformulacion_RecuperacionBayesiana.ipynb` | Exploratorio: modelo de la recuperación con regresión bayesiana |
| `Exploratorio_Recuperacion_LightGBM.ipynb` | Exploratorio: orden por saldo × (1 − recuperación predicha) con LightGBM |

## Cómo reproducir

1. Abrir cualquiera de los cuadernos en Google Colab y ejecutar todas las celdas en orden (**Entorno de ejecución → Ejecutar todas**).
2. El cuaderno obtiene la base automáticamente: usa la copia versionada en el Drive del equipo o, si no tiene acceso, la descarga del Release v1.0-avance3 de este repositorio. En los dos casos verifica la huella SHA-256 y que se carguen las 2.260.701 filas.
3. La semilla fija (2026) garantiza que el muestreo de Gibbs y el bootstrap den las mismas cifras.
4. `Proyecto_LGD_VersionadoBase.ipynb` solo se necesita para regenerar la copia versionada a partir del archivo de Kaggle.

## Librerías
## Librerías

Los cuadernos se ejecutaron en Google Colab con Python 3.13.18 y estas versiones (9 de noviembre de 2026):

| Librería | Versión | Para qué se usa |
|---|---|---|
| pandas | 2.2.3 | Cargar y manipular la base |
| pyarrow | 23.0.1 | Leer la copia versionada en formato Parquet |
| numpy | 2.4.6 | Cálculos numéricos y muestreo de Gibbs |
| matplotlib | 3.10.0 | Gráficas |
| seaborn | 0.13.2 | Gráficas del análisis exploratorio |
| scipy | 1.16.3 | Distribuciones de las priors y estadística |
| scikit-learn | 1.6.1 | Pipeline, validación cruzada y línea base |
| lightgbm | 4.6.0 | Modelo de recuperación (solo `Exploratorio_Recuperacion_LightGBM`) |

En Colab ya vienen instaladas. Fuera de Colab se instalan con:

    pip install pandas==2.2.3 pyarrow==23.0.1 numpy==2.4.6 matplotlib==3.10.0 seaborn==0.13.2 scipy==1.16.3 scikit-learn==1.6.1 lightgbm==4.6.0 kagglehub==1.0.3

## Referencias

Basel Committee on Banking Supervision. (2022). *IRB approach: Minimum requirements to use the IRB approach (CRE 36)*. Bank for International Settlements. https://www.bis.org/basel_framework/standard/cre/36/inforce/2023-01-01/published/2022-12-08

BBVA. (2025). *Pillar 3 2024*. U.S. Securities and Exchange Commission. https://www.sec.gov/Archives/edgar/data/842180/000084218025000016/pillar3report2024sec.htm

George, N. (2019). *All Lending Club loan data* [Conjunto de datos]. Kaggle. https://www.kaggle.com/datasets/wordsforthewise/lending-club

Happen Bank. (s. f.). *What do the different Note statuses mean?* Recuperado el 9 de noviembre de 2026, de https://www.happen.com/help/investing-faq/what-do-the-different-note-statuses-mean

IFRS Foundation. (s. f.). *IFRS 9 Financial Instruments*. https://www.ifrs.org/issued-standards/list-of-standards/ifrs-9-financial-instruments/

LendingClub Corporation. (2025). *Annual report on Form 10-K for the fiscal year ended December 31, 2024*. U.S. Securities and Exchange Commission. https://www.sec.gov/Archives/edgar/data/1409970/000140997025000009/lc-20241231.htm

Zhou, G., Zhang, Y., & Luo, S. (2018). P2P network lending, loss given default and credit risks. *Sustainability, 10*(4), 1010. https://doi.org/10.3390/su10041010
