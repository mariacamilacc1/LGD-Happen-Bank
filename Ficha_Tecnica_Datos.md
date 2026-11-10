# Ficha Técnica de los Datos

## Fuente

| Elemento | Valor |
|---|---|
| Conjunto de datos | *All Lending Club loan data* (George, 2019), Kaggle |
| Archivo | `accepted_2007_to_2018Q4.csv.gz` (créditos aceptados) |
| Descarga | Directa al cuaderno con `kagglehub`, con control de integridad: si se cargan menos de 2 millones de filas, el cuaderno se detiene |
| Filas del archivo | 2.260.701 (33 son filas de resumen y se eliminan) |
| Periodo de originación | 2007–2018 |
| Fecha de corte | Marzo de 2019 (último pago registrado en la base) |
| Población del estudio | 269.320 créditos castigados (*Charged Off*, incluidos los 761 castigados fuera de política); se excluyen los 40 en estado *Default* |
| Base del análisis | 196.403 créditos tras la ventana de maduración de 15 meses; 196.364 en la base de ajuste |

## Inventario de datos: disponibilidad, calidad y brechas

Una fila por variable que el análisis necesita, incluidas las que no están disponibles. Los porcentajes de faltantes se calculan sobre los 269.320 créditos castigados.

| Variable necesaria | Nombre en el código | ¿Disponible? | Fuente y campo | Cobertura / periodo | % faltantes | Problemas de calidad detectados | ¿Existe ANTES de decidir? | Qué se hará si falta |
|---|---|---|---|---|---|---|---|---|
| Identificador del crédito | `id_prestamo` | Sí | Kaggle, `id` | 2007–2018 | 0 % | 33 filas de resumen al final del archivo, con id no numérico; sin créditos repetidos | Sí (originación) | Eliminar las filas de resumen |
| Identificador del prestatario | — | No | `member_id` | — | 100 % (vacío) | No permite saber si un prestatario tiene varios créditos | — | Partición por tiempo; se declara como limitación |
| Monto financiado | `monto` | Sí | Kaggle, `funded_amnt` | 2007–2018 | 0 % | Es el monto original, no el saldo al default | Sí (originación) | Excluir el registro |
| Capital pagado | `capital_pagado` | Sí | `total_rec_prncp` | 2007–2018 | 0 % | Incluye pagos hechos antes del default | Parcial | Excluir el registro |
| Recuperaciones posteriores | `recuperacion` | Sí | `recoveries` | 2007–2018 | 0 % | Recuperaciones tardías no registradas en los años recientes (censura del target) | No (posterior) | Excluir el registro |
| Costo de cobranza | `comision` | Sí | `collection_recovery_fee` | 2007–2018 | 0 % | Mismo problema de censura | No (posterior) | Excluir o usar LGD bruta |
| Estado del préstamo | `estado` | Sí | `loan_status` | 2007–2018 | 0 % | Hay varias etiquetas de pérdida | Define el default | Excluir el registro |
| Tasa de interés | `tasa` | Sí | `int_rate` | 2007–2018 | 0 % | Rango 5,31–30,99 %; se asigna según la calificación, por eso es colineal con grade | Sí (originación) | Excluir el registro. No entra al modelo por multicolinealidad (VIF de 9,7) |
| Ingreso anual | `ingreso` | Sí | `annual_inc` | 2007–2018 | 0 % | 68 registros con ingreso 0 (no permiten el logaritmo) y valores extremos (máx. 9.500.000) | Sí (autodeclarado en la solicitud) | Excluir del ajuste los registros con ingreso 0 |
| Relación deuda/ingreso | `dti` | Sí | `dti` | 2007–2018 | 0,026 % (71 registros) | 143 registros con dti > 100 (máx. 999, atípicos) | Sí (originación) | Excluir del ajuste los faltantes y los valores mayores a 100 |
| Calificación de riesgo | `calificacion` | Sí | `grade` | 2007–2018 | 0 % | 7 categorías (A–G); G tiene solo 4.632 casos (1,7 %) | Sí (originación) | Excluir el registro |
| Plazo | `plazo` | Sí | `term` | 2007–2018 | 0 % | Dos valores: 36 y 60 meses (tras la ventana, 120.341 y 76.062) | Sí (originación) | Excluir el registro |
| Propósito del crédito | `proposito` | Sí | `purpose` | 2007–2018 | 0 % | 14 categorías muy desbalanceadas: debt_consolidation tiene 61,4 % y educational solo 88 casos; se agrupan las pequeñas | Sí (solicitud) | Excluir el registro |
| Fecha de originación | `fecha_originacion` | Sí | `issue_d` | 2007–2018 | 0 % | Viene como texto mes-año; el 68,8 % de los defaults se originó en 2014–2016 | Sí (originación) | Excluir el registro |
| Fecha del último pago | `fecha_ultimo_pago` | Sí | `last_pymnt_d` | 2007–2019 | 0,9 % (2.325 créditos que nunca pagaron) | Viene como texto mes-año; se usa para aproximar la fecha de default | Sí (al default) | Usar la fecha de originación |
| Fecha de default | `fecha_default` | No, se aproxima | `last_pymnt_d` o, si falta, `issue_d` | 2007–2019 | — | Aproximación contrastada con la fecha estimada por cuotas pagadas: 99,4 % con diferencia de seis meses o menos | Sí (al default) | Se aproxima en todos los registros |
| Cuota mensual | `cuota` | Sí | `installment` | 2007–2018 | 0 % | Solo se usa para el contraste de la fecha de default | Sí (originación) | — |
| Total pagado | `total_pagado` | Sí | `total_pymnt` | 2007–2018 | 0 % | Incluye las recuperaciones; solo se usa para el contraste de la fecha de default | No (posterior) | — |
| Utilización del crédito | — | Sí | `revol_util` | 2007–2018 | 0,07 % (130 registros de la base de ajuste) | Correlación con la pérdida de −0,02 | Sí (originación) | No entra al modelo |
| Antigüedad laboral | — | Sí | `emp_length` | 2007–2018 | 7,85 % (21.150 registros) | Es la única con faltantes importantes | Sí (solicitud) | No entra al modelo; se declara como limitación en 7.2 |
| Severidad de la pérdida (LGD) | `lgd` | No, se construye | Calculada con funded_amnt, total_rec_prncp, recoveries y collection_recovery_fee | 2007–2018 | 0 % | Hereda la censura de recoveries y collection_recovery_fee | No (es el resultado que se quiere estimar) | Excluir el registro |

**Regla.** Un dato solo sirve para una decisión si existe antes del momento en que la decisión se toma. Usar información posterior al resultado —o que lo contiene— es fuga de datos, y produce análisis que funcionan en el archivo y fallan en la realidad.

## Traducción de las variables

El cuaderno carga solo las 16 columnas que necesita (de las 151 de la base) y las renombra en español.

| Campo en Kaggle | Nombre en el código | Qué significa | Para qué se usa |
|---|---|---|---|
| `id` | `id_prestamo` | Identificador del crédito | Detectar filas que no son créditos y duplicados |
| `loan_status` | `estado` | Estado del crédito | Seleccionar los créditos castigados |
| `funded_amnt` | `monto` | Monto desembolsado | Denominador de la LGD y predictora |
| `total_rec_prncp` | `capital_pagado` | Capital que el cliente alcanzó a pagar | Fórmula de la LGD |
| `recoveries` | `recuperacion` | Lo recuperado después del castigo | Fórmula de la LGD |
| `collection_recovery_fee` | `comision` | Lo que cobró la agencia de cobranza | Fórmula de la LGD |
| `int_rate` | `tasa` | Tasa de interés (%) | Revisión de multicolinealidad (no entra al modelo) |
| `annual_inc` | `ingreso` | Ingreso anual declarado | Predictora (en log2) |
| `dti` | `dti` | Relación deuda/ingreso (%) | Predictora |
| `grade` | `calificacion` | Calificación de riesgo (A a G) | Predictora |
| `term` | `plazo` | Plazo (36 o 60 meses) | Predictora |
| `purpose` | `proposito` | Para qué se pidió el crédito | Predictora |
| `issue_d` | `fecha_originacion` | Mes en que se otorgó el crédito | Partición temporal |
| `last_pymnt_d` | `fecha_ultimo_pago` | Mes del último pago | Aproximar la fecha de default |
| `installment` | `cuota` | Cuota mensual | Contraste de la fecha de default |
| `total_pymnt` | `total_pagado` | Total pagado, incluidas las recuperaciones | Contraste de la fecha de default |

**Variables construidas en el código**

| Nombre en el código | Cómo se construye |
|---|---|
| `lgd` | ((monto − capital_pagado) − (recuperacion − comision)) / monto, acotada entre 0 y 1 |
| `fecha_default` | fecha_ultimo_pago; si el crédito nunca pagó, fecha_originacion |
| `meses_desde_default` | Meses entre la fecha de default y la fecha de corte (marzo de 2019) |
| `anio_originacion` | Año de fecha_originacion |

**Categorías traducidas**

| Campo | Valor original | En el código |
|---|---|---|
| `estado` | Charged Off | Castigado |
| `estado` | Does not meet the credit policy. Status:Charged Off | Castigado (fuera de política) |
| `estado` | Fully Paid | Pagado totalmente |
| `estado` | Does not meet the credit policy. Status:Fully Paid | Pagado (fuera de política) |
| `estado` | Current | Al día |
| `estado` | In Grace Period | En periodo de gracia |
| `estado` | Late (16-30 days) | En mora (16-30 días) |
| `estado` | Late (31-120 days) | En mora (31-120 días) |
| `estado` | Default | En incumplimiento |
| `plazo` | 36 months / 60 months | 36 meses / 60 meses |
| `proposito` | debt_consolidation | Consolidación de deudas |
| `proposito` | credit_card | Tarjeta de crédito |
| `proposito` | home_improvement | Mejoras del hogar |
| `proposito` | major_purchase | Compra importante |
| `proposito` | small_business | Pequeño negocio |
| `proposito` | car | Vehículo |
| `proposito` | medical | Gastos médicos |
| `proposito` | moving | Mudanza |
| `proposito` | vacation | Vacaciones |
| `proposito` | house | Vivienda |
| `proposito` | wedding | Boda |
| `proposito` | renewable_energy | Energía renovable |
| `proposito` | educational | Educación |
| `proposito` | other | Otro |

## Estados del crédito en `loan_status`

| Estado en `loan_status` | Días sin estar al día | Significado |
|---|---|---|
| In Grace Period | 1–15 | Periodo de gracia |
| Late (16-30 days) | 16–30 | Primer tramo de mora |
| Late (31-120 days) | 31–120 | Mora avanzada |
| Default | 121 o más | Incumplimiento |
| Charged Off | A más tardar ~150 | Castigo: pérdida reconocida |

Fuente: Happen Bank (s. f.). Según su definición, un crédito pasa a *Default* cuando lleva más de 120 días sin estar al día, y se castiga cuando ya no se esperan más pagos, por lo general a más tardar 30 días después de entrar en *Default*.
