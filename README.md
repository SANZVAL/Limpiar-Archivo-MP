# Limpieza de Archivo Mercado Pago

Aplicación web (HTML + JS, sin servidor) para procesar, limpiar y analizar archivos de liquidaciones exportados desde Mercado Pago.

Detecta automáticamente el formato del archivo, separa los registros válidos de los que deben excluirse, normaliza la estructura de salida y calcula los montos correspondientes.

---

## 🚀 Qué hace

La herramienta permite:

- Subir un archivo `.csv`, `.txt` o `.xlsx`
- Detectar automáticamente el formato de origen
- Aplicar las reglas de limpieza vigentes (ver más abajo)
- Separar el resultado en dos archivos:
  - ✅ `{nombre}_limpio.csv`
  - ❌ `{nombre}_eliminado.csv` (solo si hubo registros para eliminar)
- Calcular y mostrar en pantalla:
  - Cantidad de filas totales, limpias y eliminadas
  - Monto limpio y monto eliminado (ARS)
  - Detalle de eliminados por motivo
  - Detalle de normalización de TRANSACTION_TYPE

Todo el procesamiento se realiza en el navegador (no se envía información a ningún servidor).

---

## 📁 Uso

1. Subir o arrastrar el archivo
2. El sistema lo procesa automáticamente
3. Revisar el resumen en pantalla
4. Descargar `{nombre}_limpio.csv` y, si corresponde, `{nombre}_eliminado.csv`

---

## 📂 Formatos soportados

| Formato | Descripción | Separador | Comillas |
|---|---|---|---|
| 1 | CSV oficial de Mercado Pago | coma (`,`) | solo `EXTERNAL_REFERENCE` y `METADATA` |
| 2 | CSV o XLSX exportado por agentes (vía alternativa) | punto y coma (`;`) o binario Excel | sin comillas |
| 3 | CSV con la fila completa entre comillas | coma (`,`), fila envuelta en `"..."` | comillas dobles internas escapadas (`""`) |

La detección es automática y no requiere que el usuario indique el formato. Los `.xlsx` binarios se leen con [SheetJS](https://cdnjs.cloudflare.com/ajax/libs/xlsx/) directamente desde el navegador.

---

## 🧠 Reglas de limpieza

### Grupo 1 — Prefijo de EXTERNAL_REFERENCE

| Criterio | Motivo |
|---|---|
| Comienza con `Venta` | Venta presencial (POS) |
| Comienza con `INSTORE` | INSTORE (QR/local) |
| Comienza con `MP-Q` | Registro MP-Q |
| Comienza con `MELIPAYMENTS-COLLECTIONATTEMPT` | Intento de cobro MP |
| Referencia vacía seguida de un campo que empieza con `88 ` | Registro inválido (,88) |
| Comienza con `20000` | 🛒 Compras COTESMA Saldo de MP |

### Grupo 2 — EXTERNAL_REFERENCE vacío o nulo

| Condición | Motivo |
|---|---|
| Vacío + `PAYMENT_METHOD_TYPE=bank_transfer` + `PAYMENT_METHOD=interop_transfer` | 🏦 Transf. CTA MP Cotesma |
| Vacío + cualquier otro método de pago | ⚠️ Anomalía - Controlar (requiere revisión manual) |

### Grupo 3 — TRANSACTION_AMOUNT negativo

Se elimina y se etiqueta como 🔄 Reverso, **excepto** si la fila ya fue clasificada por el Grupo 1 bajo el criterio `20000` (ese caso conserva su motivo original, ya que un monto negativo ahí es esperado).

### Normalización de TRANSACTION_TYPE

En el archivo `_limpio.csv`, la columna `TRANSACTION_TYPE` siempre queda como `SETTLEMENT`, sin excepción. Si una fila trae otro valor original, se conserva la fila pero se fuerza el valor, y se informa en el resumen cuántas filas se normalizaron y desde qué valor.

---

## 📄 Estructura de los archivos de salida

- Mismo separador (coma) y mismas columnas/orden que el archivo oficial de Mercado Pago.
- Solo `EXTERNAL_REFERENCE` y `METADATA` van entre comillas dobles.
- `_limpio.csv`: no agrega columnas nuevas.
- `_eliminado.csv`: agrega la columna `MOTIVO_ELIMINACION` al final, con el motivo correspondiente de las tablas arriba.
- Si no se elimina ningún registro, no se genera `_eliminado.csv`.

---

## ⚙️ Procesamiento de texto (CSV)

El parser maneja:

- Filas con comillas solo en `EXTERNAL_REFERENCE`/`METADATA` (Formato 1)
- Filas sin comillas, separadas por `;` (Formato 2)
- Filas completas entre comillas con comillas internas escapadas (`""`) (Formato 3)
- Como respaldo, si el número de columnas no coincide con el esperado (por ejemplo, un `METADATA` con comillas internas mal escapadas), se intenta aislar el bloque `[...]` de `METADATA` antes de partir el resto de la fila. Ninguna fila se descarta por este motivo.

---

## ⚠️ Limitaciones conocidas

- El criterio `,88`/`;88` del Grupo 1 se interpreta como: `EXTERNAL_REFERENCE` vacío y el campo inmediatamente siguiente comienza con `88 `. Si en la práctica el patrón real difiere, avisar para ajustar la regla.
- No maneja saltos de línea dentro de una celda en archivos de texto (si el CSV tiene ese caso, conviene usar la variante XLSX).
- Se recomienda validar con un archivo real de cada formato antes de usar en producción, ya que las reglas no pudieron probarse contra datos reales de Mercado Pago al momento de esta actualización.

---

## 🖥️ Interfaz

- Drag & drop de archivos
- Feedback visual del archivo cargado
- Resumen con desglose por motivo de eliminación
- Aviso destacado cuando hay registros "Anomalía - Controlar"
- Descarga directa de resultados

---

## 🛠️ Tecnologías

- HTML / CSS
- JavaScript (Vanilla)
- [SheetJS](https://cdnjs.cloudflare.com/ajax/libs/xlsx/) (solo para lectura de `.xlsx`, cargado desde CDN)

---

## 📌 Autor

SANZVAL

## URL

https://sanzvalb.github.io/Limpiar-Archivo-MP/csv_cleaner_html.html
