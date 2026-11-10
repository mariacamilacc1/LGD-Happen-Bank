# Versionado de la Base de Datos

| Elemento | Valor |
|---|---|
| Conjunto de datos | *All Lending Club loan data* (George, 2019), Kaggle (`wordsforthewise/lending-club`) |
| Licencia | CC0: Public Domain |
| Versión del conjunto en Kaggle | No identificada: la descarga se hizo desde la caché de Kaggle en Colab, que no indica el número de versión. El identificador del archivo es su huella SHA-256 |
| Archivo de origen | `accepted_2007_to_2018Q4.csv.gz` |
| SHA-256 del archivo de origen | `55c16f75120f897683f02e7aabcf080d0e4a20c4832feb1d592cfa941bd62a2d` |
| Fecha de descarga original del proyecto | 26 de octubre de 2026 |
| Fecha de creación de la copia versionada | 9 de noviembre de 2026 (10 de noviembre en la hora UTC del servidor de Colab) |
| Copia versionada | `lending_club_accepted_2007_2018Q4.parquet` (Parquet, compresión zstd, 58,6 MB) |
| SHA-256 de la copia versionada | `e964f05b0ad2024931e436630f586e9790dc47ff45242afa53860ec2b974a8d8` |
| Filas | 2.260.701 (incluidas las 33 filas de resumen del archivo original) |
| Columnas | 22: id, loan_status, funded_amnt, total_rec_prncp, recoveries, collection_recovery_fee, int_rate, annual_inc, dti, grade, term, purpose, issue_d, last_pymnt_d, installment, total_pymnt, revol_util, fico_range_low, fico_range_high, emp_length, home_ownership, verification_status |
| Ubicación pública | Release [`v1.0-avance3`](https://github.com/mariacamilacc1/LGD-Happen-Bank/releases/tag/v1.0-avance3) de este repositorio |
| Ubicación del equipo | Google Drive: `MyDrive/LGD_Base_Versionada/` |

## Cómo la usan los cuadernos

Los cuatro cuadernos (`Proyecto_LGD_NotebookFinal`, `Proyecto_LGD_VentanaMaduracion`, `Exploratorio_Reformulacion_RecuperacionBayesiana` y `Exploratorio_Recuperacion_LightGBM`) buscan la base en este orden:

1. La copia versionada en el Google Drive del equipo.
2. La copia publicada en el Release `v1.0-avance3` de GitHub, que cualquier persona puede descargar.
3. El archivo original en Kaggle.

En los tres casos calculan la huella SHA-256 del archivo, la comparan con la registrada en esta tabla e informan si coincide, y verifican que se carguen las 2.260.701 filas.

## Cómo verificar que la copia no cambió

Calcular el SHA-256 del archivo Parquet y compararlo con el de esta tabla. Si no coincide, la copia fue modificada y debe regenerarse con `Proyecto_LGD_VersionadoBase.ipynb` a partir del archivo de origen, cuya huella también está registrada.

## Uso de los datos

George (2019) publica el conjunto en Kaggle bajo la licencia CC0: Public Domain, que permite usarlo, copiarlo y redistribuirlo sin restricciones; por eso la copia versionada se publica en el Release del repositorio. Se tienen en cuenta dos advertencias del autor en la descripción del conjunto:

- Los datos provienen originalmente de LendingClub (hoy Happen Bank), que restringió su descarga con términos de uso, y el autor advierte que podrían pedir retirar el conjunto de Kaggle. La copia versionada permite reproducir el análisis aunque eso ocurra.
- El autor limpió ligeramente los datos: quitó el símbolo de porcentaje de `int_rate` y `revol_util` y los convirtió a números.
