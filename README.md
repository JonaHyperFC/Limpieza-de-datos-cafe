# Limpieza y transformación de datos: Cafe Sales (Dirty Data)

Práctica individual de *Análisis y visualización de la información*.

**Objetivo:** aplicar técnicas de limpieza y transformación de datos con Python sobre una base diseñada para contener problemas de calidad.

**Dataset:** [Cafe Sales – Dirty Data for Cleaning Training](https://www.kaggle.com/datasets/ahmedmohamed2003/cafe-sales-dirty-data-for-cleaning-training) (Kaggle), archivo `dirty_cafe_sales.csv`: 10,000 transacciones de una cafetería con las columnas `Transaction ID`, `Item`, `Quantity`, `Price Per Unit`, `Total Spent`, `Payment Method`, `Location` y `Transaction Date`.

## Contenido del repositorio

| Archivo | Descripción |
|---|---|
| `limpieza_cafe_sales.ipynb` | Notebook con todo el procedimiento (código, salidas y justificaciones) |
| `cafe_sales_clean.csv` | Base limpia exportada (9,974 filas) |
| `tabla_resumen.md` / `tabla_resumen.csv` | Tabla de problemas, acciones y justificaciones |
| `README.md` | Este documento |

Para reproducirlo: descargar `dirty_cafe_sales.csv` de Kaggle, colocarlo en la misma carpeta que el notebook y ejecutarlo completo (requiere `pandas` y `numpy`).

## Procedimiento

1. **Carga e inspección.** Se lee el CSV con todo como texto (`dtype=str`) para no perder marcadores como `ERROR` o `UNKNOWN`. Se revisan tipos, nulos, duplicados y frecuencias de las columnas categóricas.
2. **Limpieza básica.**
   - Se estandarizan los nombres de columnas y se eliminan espacios sobrantes.
   - Los marcadores `ERROR`, `UNKNOWN` y vacíos se convierten a `NaN`, porque son datos faltantes y no valores reales.
   - Se revisan duplicados e IDs repetidos.
   - Se convierten los tipos: numérico para cantidad, precio y total, y fecha para `transaction_date`.
   - Se buscan valores imposibles (≤ 0).
3. **Recuperación de datos sin inventar información.** Se rellenan faltantes solo cuando hay una relación determinista entre columnas, evitando medias y medianas que sesgarían los montos:
   - Precio a partir del ítem, ya que cada producto tiene precio fijo.
   - `total = cantidad × precio`, despejando la variable faltante (la cantidad solo si resulta entera).
   - Ítem a partir del precio, únicamente cuando ese precio pertenece a un solo producto. Los precios 3.0 (Cake/Juice) y 4.0 (Sandwich/Smoothie) son ambiguos y no se imputan.
4. **Faltantes no recuperables.**
   - Se eliminan las filas sin cantidad, precio o total recuperable (26 filas).
   - Las categorías `item`, `payment_method` y `location` se rellenan con `Unknown`.
   - Las fechas faltantes se mantienen como `NaT`.
5. **Transformaciones.** `quantity` pasa a entero y se agregan las columnas `year`, `month` y `weekday`.
6. **Validación y exportación.** Se verifica que el ID sea único, que `cantidad × precio = total` en todas las filas y que no queden nulos en las columnas clave. Después se exporta la base limpia.

## Resultado

- Filas originales: **10,000** → filas finales: **9,974**.
- Sin nulos en `item`, `quantity`, `price_per_unit`, `total_spent`, `payment_method` ni `location`.
- 460 fechas siguen faltantes (y, por tanto, `year`, `month` y `weekday`), a propósito.
- **Limitación:** `payment_method` (3,168) y `location` (3,952) conservan muchos valores `Unknown`, por lo que los análisis por esas variables deben interpretarse con cautela.

## Tabla resumen de problemas y decisiones

| Problema encontrado | Registros afectados | Acción realizada | Justificación |
|---|---|---|---|
| Faltantes en item (636 'ERROR'/'UNKNOWN' + 333 nulos) | 969 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Faltantes en quantity (341 'ERROR'/'UNKNOWN' + 138 nulos) | 479 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Faltantes en price_per_unit (354 'ERROR'/'UNKNOWN' + 179 nulos) | 533 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Faltantes en total_spent (329 'ERROR'/'UNKNOWN' + 173 nulos) | 502 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Faltantes en payment_method (599 'ERROR'/'UNKNOWN' + 2579 nulos) | 3178 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Faltantes en location (696 'ERROR'/'UNKNOWN' + 3265 nulos) | 3961 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Faltantes en transaction_date (301 'ERROR'/'UNKNOWN' + 159 nulos) | 460 | Unificar todos como NaN | Son marcadores de dato faltante, no valores reales; un solo criterio simplifica la imputación. |
| Columnas con tipo incorrecto (todo texto) | 4 | Cast a numérico (quantity, price_per_unit, total_spent) y datetime (transaction_date) | Permite operar, ordenar y agregar correctamente. |
| Price Per Unit faltante con Item conocido | 479 | Imputar con el precio del menú para ese ítem | Cada producto tiene precio fijo; la recuperación es determinista. |
| Total Spent faltante | 479 | Calcular quantity × price_per_unit | Relación aritmética exacta. |
| Quantity faltante | 456 | Calcular total_spent ÷ price_per_unit | Se recupera solo si el resultado es entero. |
| Price Per Unit faltante (con total y cantidad) | 48 | Calcular total_spent ÷ quantity | Relación aritmética exacta. |
| Item faltante con precio único en el menú | 489 | Inferir ítem desde el precio | Solo cuando el precio identifica un único producto; los precios ambiguos no se imputan. |
| Filas sin monto/cantidad/precio recuperable | 26 | Eliminar filas | Sin variables numéricas clave no aportan al análisis; imputar con media/mediana sesgaría los totales. |
| item faltante sin recuperación posible | 474 | Rellenar con 'Unknown' | Se conserva la venta (monto válido) y la categoría desconocida queda explícita. |
| payment_method faltante sin recuperación posible | 3168 | Rellenar con 'Unknown' | Se conserva la venta (monto válido) y la categoría desconocida queda explícita. |
| location faltante sin recuperación posible | 3952 | Rellenar con 'Unknown' | Se conserva la venta (monto válido) y la categoría desconocida queda explícita. |
| Fecha faltante | 460 | Mantener como NaT | Inventar fechas distorsionaría series de tiempo; se excluyen solo en análisis temporales. |
