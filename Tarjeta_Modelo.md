# Tarjeta del Modelo

Formato basado en Mitchell et al. (2019), *Model Cards for Model Reporting*.

## 1. Detalles del modelo

| Elemento | Valor |
|---|---|
| Nombre | Regresión lineal bayesiana de la severidad de la pérdida (LGD) |
| Equipo | Daira Julieth Campos V., María Camila Cárdenas C. y Alejandro José Medina C. (Universidad Externado de Colombia) |
| Versión | v1.0-avance3 (10 de noviembre de 2026) |
| Tipo de modelo | Regresión lineal múltiple bayesiana con priors semiconjugadas: Normal para los coeficientes y Gamma-Inversa para la varianza |
| Estimación | Muestreo de Gibbs: 4 cadenas de 6.000 iteraciones, se descartan las 1.000 primeras de cada una; R-hat máximo de 1,0002 |
| Variable objetivo | LGD = ((monto − capital pagado) − (recuperación − comisión)) / monto, acotada entre 0 y 1 |
| Predictoras | Calificación de riesgo (A a G), plazo (36 o 60 meses), propósito (los de menos del 1 % agrupados en "Otro"), monto, ingreso (en log2) y relación deuda/ingreso, todas centradas |
| Código | `Proyecto_LGD_NotebookFinal.ipynb` (pipeline de scikit-learn con la preparación y el modelo) |
| Datos | Copia versionada en el Release `v1.0-avance3` (ver `Versionado_Base_Datos.md` y `Ficha_Tecnica_Datos.md`) |

## 2. Uso previsto

- **Uso previsto:** cuantificar cómo cambia la pérdida de los créditos castigados según sus características (calificación, plazo, propósito, monto, ingreso y relación deuda/ingreso), con la incertidumbre de cada estimación, como insumo de análisis para el área de cobranza.
- **Usuarios previstos:** analistas de riesgo y de cobranza.
- **Usos no previstos:**
  - Priorizar la cobranza de forma automática: en la prueba de 2015 el orden del modelo captura el 39,2 % de la pérdida en el 20 % priorizado, por debajo del 40,6 % del orden por saldo al default.
  - Calcular la LGD regulatoria: la LGD del proyecto se mide sobre el monto financiado y no sobre la exposición al default, sin descuento ni costos indirectos (Basel Committee on Banking Supervision, 2022, CRE 36).
  - Tomar decisiones sobre personas (aprobación, precio o condiciones de un crédito).
  - Aplicarlo a créditos originados después de 2015 o a otras entidades.

## 3. Factores

- **Factores evaluados:** calificación de riesgo y plazo, que son las características más asociadas con la pérdida.
- **Subgrupos de equidad:** el error por monto (tercio menor frente a tercio mayor, umbral de 15 % en el MAE) y por propósito (umbral de 1,2 veces el RMSE global), fijados en el A2-Paso 7.3, está pendiente como condición C2 del A3-Paso 9.

## 4. Métricas

| Métrica | Por qué se usa | Línea base |
|---|---|---|
| Captura@20 %: proporción de la pérdida total en USD que concentra el 20 % de créditos priorizados | Es la decisión del proyecto: cobranza solo puede atender el 20 % | Orden por saldo al default (40,6 %); criterio de éxito de al menos 40,9 % |
| RMSE de la LGD | Mide el error de la estimación de la pérdida | LGD promedio de 2007–2014 para todos los créditos (RMSE de 0,2059); criterio de éxito de una mejora de al menos 10 % |

## 5. Datos de entrenamiento y de evaluación

| | Créditos | Periodo |
|---|---|---|
| Entrenamiento | 75.175 | Originados entre 2007 y 2014 |
| Validación cruzada temporal (ventana expansiva) | Tres pliegues | Validan 2012, 2013 y 2014 |
| Prueba | 64.846 | Originados en 2015 |

- **Población:** 269.320 créditos castigados de LendingClub (hoy Happen Bank), de los que 196.403 cumplen la ventana de maduración de 15 meses desde el default.
- **Excluidos del modelo:** 2016–2017 (cohortes incompletas por la ventana) y 39 créditos con ingreso de 0 o relación deuda/ingreso vacía o mayor a 100.
- **Preparación:** se aprende solo con el entrenamiento (centrado, logaritmo del ingreso, agrupación de propósitos y dummies).

## 6. Resultados cuantitativos (prueba de 2015)

| Resultado | Valor | Comparación |
|---|---|---|
| RMSE | 0,1855 [0,1849; 0,1860] | 0,2059 de la línea base |
| Mejora del RMSE | 9,9 % [9,63 %; 10,16 %], P(mejora ≥ 10 %) = 0,23 | Criterio de 10 %: no se cumple |
| Captura@20 % | 39,2 % | 40,6 % del orden por saldo; 41,2 % como máximo posible: no se cumple |
| Diferencia de captura frente al saldo | −1,4 pp [−1,46; −1,33] | — |
| Mejora del RMSE en la validación cruzada | 2,7 % (2012), 4,0 % (2013) y 7,6 % (2014) | — |
| Efecto de la calificación G frente a A | +13,2 pp de pérdida [11,9; 14,6] | Sin ajustar, 23,6 pp |
| Efecto del plazo de 60 frente a 36 meses | +10,9 pp de pérdida | — |
| Costo asimétrico (penalización 5:1) | Factor k = 1,32; la línea base ajustada logra lo mismo (diferencia de −0,3 %) | Supuesto del equipo, no del banco |

## 7. Consideraciones éticas

- **Sesgo de selección:** la ventana excluye 72.917 créditos (27,1 %). Se parecen a los incluidos en monto, ingreso y tasa (diferencias menores a 8 %), pero tienen menos créditos de calificación E a G (16,1 % frente a 22,2 %) y, en 2015, más plazo a 60 meses (58,0 % frente a 42,1 %), porque cayeron en default más tarde.
- **Independencia:** `member_id` está vacío en toda la base, así que un prestatario podría tener varios créditos, incluso a ambos lados de la partición.
- **Variables sustitutas:** el propósito y la calificación pueden relacionarse con la condición socioeconómica del prestatario.
- **Datos:** son datos públicos de un tercero (Kaggle, licencia CC0: Public Domain), no la fuente oficial de Happen Bank.

## 8. Advertencias y recomendaciones

- La likelihood Normal no está acotada: el 3,4 % de las LGD simuladas cae fuera de [0, 1] y la cobertura del intervalo predictivo es de 97,9 % frente al 95 % nominal. Se recomienda una regresión Beta inflada en 0 y 1.
- Los resultados aplican a créditos originados entre 2007 y 2015 con al menos 15 meses de recuperación.
- Dictamen del proyecto: **SE REFORMULA**. Mejorar la priorización requiere datos del proceso de cobranza (contacto, acuerdos de pago, tiempos de gestión), que se solicitarán a Happen Bank (condición C1).

## Referencias

Basel Committee on Banking Supervision. (2022). *IRB approach: Minimum requirements to use the IRB approach (CRE 36)*. Bank for International Settlements. https://www.bis.org/basel_framework/standard/cre/36/inforce/2023-01-01/published/2022-12-08

Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., Spitzer, E., Raji, I. D., & Gebru, T. (2019). Model cards for model reporting. En *Proceedings of the Conference on Fairness, Accountability, and Transparency* (pp. 220–229). ACM. https://doi.org/10.1145/3287560.3287596
