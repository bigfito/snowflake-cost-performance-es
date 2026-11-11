# Recetas SQL para analizar costo y desempeño en Snowflake

Notebooks de Snowflake con consultas SQL para revisar, en la cuenta donde se ejecutan, el costo de cómputo, el costo de almacenamiento, el desempeño de las consultas en los virtual warehouses, el etiquetado con tags, los resource monitors y, para partners, las vistas de facturación del esquema `SNOWFLAKE.BILLING`.

Están escritos para Snowflake Notebooks con Snowpark y toman como referencia la [documentación oficial de Snowflake](https://docs.snowflake.com). Este repositorio no trae resultados de una cuenta de ejemplo: cada consulta lee el uso de la cuenta en la que se corre.

## Notebooks

| Notebook | Pregunta de negocio | Vistas | Rol |
| --- | --- | --- | --- |
| [`01 - Compute Cost Recipes.ipynb`](01%20-%20Compute%20Cost%20Recipes.ipynb)<br>Adolfo Orozco, 2025-11-06 | ¿Cuántos créditos consumen los warehouses y el resto de servicios de cómputo, y qué días se separan del promedio móvil? | `SNOWFLAKE.ACCOUNT_USAGE`: `WAREHOUSE_METERING_HISTORY`, `METERING_DAILY_HISTORY`, `METERING_HISTORY`, `QUERY_HISTORY`, `SESSIONS`, `AUTOMATIC_CLUSTERING_HISTORY`, `SEARCH_OPTIMIZATION_HISTORY`, `MATERIALIZED_VIEW_REFRESH_HISTORY`, `QUERY_ACCELERATION_HISTORY`, `COPY_HISTORY`, `PIPE_USAGE_HISTORY`, `SERVERLESS_ALERT_HISTORY`, `SERVERLESS_TASK_HISTORY`, `REPLICATION_GROUP_USAGE_HISTORY`, `DATABASE_REPLICATION_USAGE_HISTORY`, `HYBRID_TABLE_USAGE_HISTORY`, `CORTEX_ANALYST_USAGE_HISTORY`, `CORTEX_FINE_TUNING_USAGE_HISTORY`, `CORTEX_FUNCTIONS_USAGE_HISTORY`, `CORTEX_FUNCTIONS_QUERY_USAGE_HISTORY`, `CORTEX_SEARCH_DAILY_USAGE_HISTORY`, `CORTEX_SEARCH_SERVING_USAGE_HISTORY`, `DOCUMENT_AI_USAGE_HISTORY`, `CORTEX_DOCUMENT_PROCESSING_USAGE_HISTORY`, `NOTEBOOKS_CONTAINER_RUNTIME_HISTORY` | `ACCOUNTADMIN` por defecto, o un rol con `IMPORTED PRIVILEGES` sobre la base `SNOWFLAKE` |
| [`02 - Storage Cost Recipes.ipynb`](02%20-%20Storage%20Cost%20Recipes.ipynb)<br>Adolfo Orozco, 2025-11-06 | ¿Cuánto volumen hay en stages, por año y mes, y cuánto en tablas más fail-safe en los últimos 12 meses? | `SNOWFLAKE.ACCOUNT_USAGE.STAGE_STORAGE_USAGE_HISTORY`, `SNOWFLAKE.ACCOUNT_USAGE.DATABASE_STORAGE_USAGE_HISTORY` | `ACCOUNTADMIN` por defecto, o un rol con `IMPORTED PRIVILEGES` sobre la base `SNOWFLAKE` |
| [`03 - Data Transfer Cost Recipes.ipynb`](03%20-%20Data%20Transfer%20Cost%20Recipes.ipynb)<br>Adolfo Orozco, 2025-11-06 | ¿En qué días del último mes un warehouse tuvo consultas en cola? | `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY` | `ACCOUNTADMIN` por defecto, o un rol con `IMPORTED PRIVILEGES` sobre la base `SNOWFLAKE` |
| [`04 - Compute Performance Recipes.ipynb`](04%20-%20Compute%20Performance%20Recipes.ipynb)<br>Adolfo Orozco, 2025-11-06; créditos: Juan Manuel Díaz | ¿Qué consultas derraman a almacenamiento remoto, qué warehouses parecen sobredimensionados y cuáles podrían usar Query Acceleration? | `SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_LOAD_HISTORY`, `QUERY_HISTORY`, `WAREHOUSE_EVENTS_HISTORY`, `QUERY_ACCELERATION_ELIGIBLE` | `ACCOUNTADMIN` por defecto, o un rol con `IMPORTED PRIVILEGES` sobre la base `SNOWFLAKE` |
| [`05 - Tags y Resource Monitors.ipynb`](05%20-%20Tags%20y%20Resource%20Monitors.ipynb)<br>Jonathan Sierra, 2026-01-05 | ¿Cómo se reparte el crédito por tag, qué warehouses no tienen tag o resource monitor, y qué días se alejan del consumo habitual? | `SNOWFLAKE.ACCOUNT_USAGE.TAGS`, `TAG_REFERENCES`, `WAREHOUSES`, `WAREHOUSE_METERING_HISTORY`, `RESOURCE_MONITORS` | Lectura: el mismo acceso a `ACCOUNT_USAGE`. Crear tags, resource monitors y asignar el monitor de la cuenta: `ACCOUNTADMIN` o los permisos concretos que indica el cuaderno |
| [`06 - Partner Billing Views.ipynb`](06%20-%20Partner%20Billing%20Views.ipynb)<br>Jonathan Sierra, 2026-01-05 | Para un reseller, ¿cuál es el contrato, la tarifa, el saldo y el uso en moneda de cada cliente? | `SNOWFLAKE.BILLING.PARTNER_CONTRACT_ITEMS`, `PARTNER_RATE_SHEET_DAILY`, `PARTNER_REMAINING_BALANCE_DAILY`, `PARTNER_USAGE_IN_CURRENCY_DAILY`. `SNOWFLAKE.ORGANIZATION_USAGE.ACCOUNTS` aparece solo en un bloque que el cuaderno imprime y no ejecuta | `ACCOUNTADMIN`. El cuaderno pide además cuenta de organización, o una cuenta con el rol `ORGADMIN` habilitado |

El archivo `03 - Data Transfer Cost Recipes.ipynb` lleva "Data Transfer" en el nombre. La única consulta que contiene lee `WAREHOUSE_LOAD_HISTORY` (consultas en ejecución frente a consultas en cola, con `SUM(AVG_QUEUED_LOAD) > 0`). El filtro de fechas es `DATEADD(MONTH, -1, CURRENT_DATE())`. Es la misma consulta que la sección 4 de `04 - Compute Performance Recipes.ipynb`. No consulta una vista de data transfer.

En el cuaderno 02 el texto habla de terabytes. Las columnas calculadas usan `POWER(1024, 4)` y se llaman `total_stage_volume_tib`, `avg_table_storage_tib`, `avg_failsafe_storage_tib` y `avg_total_storage_tib`.

## Requisitos

- Una cuenta de Snowflake y un virtual warehouse para el notebook. Un warehouse **X-SMALL** alcanza para estas consultas.
- Para los cuadernos 01 a 05, un rol que pueda leer `SNOWFLAKE.ACCOUNT_USAGE`. En la conexión local el valor por defecto de `SNOWFLAKE_ROLE` es `ACCOUNTADMIN`. Los cuadernos 01, 05 y 06 ejecutan `USE ROLE` con ese valor. Los cuadernos 02, 03 y 04 solo lo pasan al crear la sesión local; dentro de Snowflake Notebooks usan el rol de la sesión activa. También sirve un rol al que se le haya concedido `IMPORTED PRIVILEGES` sobre la base de datos `SNOWFLAKE`.
- Esas vistas no son en tiempo real. El cuaderno 05 indica una latencia de hasta 45 minutos. Según la vista, la latencia documentada de `ACCOUNT_USAGE` puede llegar a unas horas.
- El cuaderno 05, además de leer, imprime DDL de ejemplo (`CREATE DATABASE`, `CREATE TAG`, `ALTER … SET TAG`, `CREATE RESOURCE MONITOR`, `CREATE ALERT`). Esos objetos no se crean al ejecutar el notebook: el código hace `print` del SQL. Varias sentencias usan marcadores (`<WAREHOUSE_NAME>`, `<DATABASE_NAME>`, `<USERNAME>`, `<ALERT_WAREHOUSE>`, `<DEPARTMENT>`).
- El cuaderno 06 lee `SNOWFLAKE.BILLING`, un esquema para resellers y distribuidores. El propio cuaderno pide `ACCOUNTADMIN`, una cuenta de organización o `ORGADMIN` habilitado, y avisa de una latencia de hasta 24 horas. También indica que, hasta el cierre de mes, las cifras pueden cambiar por ajustes o enmiendas de contrato, y que el saldo on-demand se reinicia después del cierre (el cuaderno dice que suele ser el día 3 o 4 del mes siguiente).

## Cómo ejecutarlo

### En Snowflake Notebooks

1. Importa el archivo `.ipynb` en Snowflake Notebooks.
2. Asocia un virtual warehouse. X-SMALL es suficiente.
3. Ejecuta las celdas en orden. Ajusta antes los parámetros de la primera celda de código.

Parámetros que cambian el SQL:

| Notebook | Parámetros | Valor en el archivo |
| --- | --- | --- |
| 01 | `M_DAYS`, `QUERY_TYPE`, `LAST_N_MINUTES`, `MODEL_NAME`, `CORTEX_QUERY_ID`, `NB_NAME`, `CP_NAME` | `30`, `COPY`, `60`, `mistral-large`, `query-id`, `<example_nb_name>`, `<example_cp_name>` |
| 04 | `WH_LARGE`, `WH_SMALL` | los literales `WH_LARGE` y `WH_SMALL`; hay que sustituirlos por warehouses reales. `QAS_USER_FILTER` y `QAS_WH_FILTER` vienen vacíos y, así, no filtran usuario ni warehouse |
| 05 | `M_DAYS`, `TAG_DATABASE`, `TAG_SCHEMA`, `COST_CENTER_TAG`, `PROJECT_TAG`, `ENVIRONMENT_TAG`, `CREDIT_QUOTA_MONTHLY` | `30`, `GOVERNANCE`, `TAGS`, `COST_CENTER`, `PROJECT`, `ENVIRONMENT`, `1000`. El monitor de cuenta de ejemplo usa cuota `5000` fija en el SQL impreso |
| 06 | `M_DAYS`, `CUSTOMER_NAME` | `30` y cadena vacía. Con `CUSTOMER_NAME` vacío, la celda 8.4 no consulta: imprime el SQL. `REPORT_DATE` (`2025-12-31`) y `CURRENCY` (`USD`) están definidos y las consultas no los referencian |

En el cuaderno 01, la celda 3.3 calcula `pct_over_to_m_day_average` como `100 * (créditos del día / promedio de la ventana - 1)`. Esa ventana es `AVG(...) OVER (PARTITION BY warehouse_name ORDER BY start_date ROWS M_DAYS PRECEDING)`. La consulta conserva filas con más de 100 créditos y ese porcentaje `>= 0.5`. El comentario de la celda dice "flags >= 50% rise". El umbral escrito en el `QUALIFY` es `0.5` sobre la columna ya multiplicada por 100.

La celda de tablas híbridas comentada como "Past year" agrupa `HYBRID_TABLE_USAGE_HISTORY` sin filtro de fechas. La celda siguiente sí limita a `M_DAYS`.

### En Jupyter local

Si `get_active_session()` no está disponible, los notebooks abren una sesión de Snowpark con variables de entorno. Si no hay `SNOWFLAKE_PASSWORD`, piden la contraseña con `getpass`.

Los cuadernos 02, 03 y 04 mencionan los paquetes `snowflake-snowpark-python[pandas]` y, como opcional, `python-dotenv`. La instalación está comentada. El código de conexión usa `os.getenv` y no llama a `load_dotenv`.

| Variable | Valor por defecto en el código |
| --- | --- |
| `SNOWFLAKE_ACCOUNT` | `your_account` |
| `SNOWFLAKE_USER` | `your_user` |
| `SNOWFLAKE_PASSWORD` | vacío; en ese caso se pide por consola |
| `SNOWFLAKE_ROLE` | `ACCOUNTADMIN` |
| `SNOWFLAKE_WAREHOUSE` | `COMPUTE_WH` |
| `SNOWFLAKE_DATABASE` | `SNOWFLAKE` |
| `SNOWFLAKE_SCHEMA` | `ACCOUNT_USAGE` en los cuadernos 01 a 05; `BILLING` en el 06 |

No guardes la contraseña en el repositorio.

## Cómo interpretar los resultados

Las consultas ordenan o etiquetan filas. Lo que sigue es el criterio escrito en cada notebook, no una lectura de una cuenta concreta.

**Cómputo (01).** En warehouses, mira primero `credits_used_compute` o `credits_used_compute_sum`. En las consultas ordenadas por créditos de mayor a menor, la primera fila es el mayor consumidor de esa ventana (warehouse, tipo de consulta, pipe, objeto de clustering, vista materializada o réplica). Otras celdas ordenan por hora, por semana o devuelven el detalle de Cortex sin ese ranking. Para un día fuera de lo habitual, la celda 3.3 deja los días con `credits_used_date_wh > 100` que además superan el umbral de `pct_over_to_m_day_average` descrito arriba. En cloud services, mira `billed_cloud_services` y `percent_cloud_services`. La nota final del cuaderno pide acotar las ventanas de tiempo en los escaneos pesados.

**Almacenamiento (02).** Mira `total_stage_volume_tib` por mes y, en tablas, `avg_total_storage_tib` (promedio de `average_database_bytes` + `average_failsafe_bytes`). El cuaderno define el costo de almacenamiento como la suma de stages, tablas y fail-safe / Time Travel, cobrado por TB al mes según la región, y enlaza la [página de precios de Snowflake](https://www.snowflake.com/en/pricing-options/). El cuaderno cita, como ejemplo de esa página, 23 USD por TB/mes para AWS en US East. No trae otra acción aparte de leer esos volúmenes.

**Colas (03 y sección 4 del 04).** Mira `SUM_QUEUED_QUERIES` junto a `SUM_RUNNING_QUERIES`. El texto del cuaderno dice que los días con colas elevadas pueden sugerir activar multi-cluster o ajustar el tamaño del warehouse.

**Desempeño (04).** Mira primero `BYTES_SPILLED_TO_REMOTE_STORAGE` y el cociente contra `BYTES_SCANNED` (`SPILLING_READ_RATIO` o `SPILLING_REMOTE_READ_RATIO`). El cuaderno recomienda tuning o un warehouse mayor cuando el spilling remoto es más de 5 veces los bytes escaneados. Las secciones 5 y 8 no filtran por fecha. La sección 6 lista las 10 consultas con más spilling remoto en 45 días e incluye `TOTAL_ELAPSED_TIME_SEC`. La sección 9 lista warehouses con tamaño distinto de `X-Small`, más de 100 consultas en 3 meses y `SUM(BYTES_SPILLED_TO_LOCAL_STORAGE) = 0`, y los encabeza como potencialmente sobredimensionados. Para Query Acceleration, la consulta se queda con `ELIGIBLE_QUERY_ACCELERATION_TIME > 120` y ordena por `TOTAL_ACCELERATION_TIME_MS`. La sección 11 muestra, como ejemplo de política y sin ejecutarlo, `ALTER ACCOUNT SET STATEMENT_TIMEOUT_IN_SECONDS = 7200`.

**Tags y resource monitors (05).** Mira primero `total_credits` por `cost_center` (lo que no tiene tag queda como `SIN_TAG`) y el porcentaje `pct_of_total`. La consulta de objetos sin tag ordena por `total_credits_30d`. La de resource monitors lista warehouses con `resource_monitor` nulo o vacío. La de anomalías calcula un z-score sobre los créditos diarios de los últimos `M_DAYS` y etiqueta `ANOMALIA ALTA` si es mayor que 2 y `ANOMALIA BAJA` si es menor que -2. El resumen del cuaderno propone etiquetar los warehouses con centro de costo y ambiente, asignar monitores a la cuenta y a los warehouses de más consumo, y revisar la cuota. El DDL de ejemplo notifica al 75 % y al 90 %, suspende al 100 % y hace `SUSPEND_IMMEDIATE` al 110 %; el de cuenta notifica al 50 %, 75 % y 90 % y suspende al 100 %. Esos porcentajes están en el SQL impreso, no son un umbral medido en una cuenta.

**Facturación de partners (06).** Mira el saldo reciente (`capacity_balance + free_usage_balance + rollover_balance`) y `pct_remaining` frente al monto de contrato. La consulta de saldo bajo deja clientes por debajo del 20 % y marca `CRITICO` bajo 10 %, `BAJO` bajo 20 % y `MODERADO` bajo 50 %. El dashboard ejecutivo usa otros cortes: `CRITICO` bajo 10 %, `ALERTA` bajo 25 %, y `RENOVACION` si faltan menos de 60 días para `contract_end`. Los contratos por vencer son los que tienen `expiration_date` dentro de 90 días. El consumo on-demand que el cuaderno destaca es `on_demand_consumption_balance < 0`. La proyección de agotamiento divide el saldo entre el uso diario medio de `M_DAYS` y etiqueta `URGENTE` (menos de 30 días), `ALERTA` (menos de 60) y `MONITOREAR` (menos de 90). Esas etiquetas son reglas del SQL.

## Licencia

[MIT](LICENSE). Copyright (c) 2026 Adolfo Orozco.
