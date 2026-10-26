# Plan de implementación: gastos-cli — MVP de gastos personales

**Feature**: `001-gastos-mvp`

**Spec**: [`spec.md`](./spec.md)

**Fecha**: 2026-10-26

**Alcance**: RF-1 a RF-11.

## 1. Resumen

Implementar una CLI de un solo usuario, sin red y con estado local. El dominio se mantendrá puro y testable; la persistencia y la interfaz de terminal quedarán separadas. La solución usará Python 3.12+, solo biblioteca estándar y pytest para pruebas. El flujo principal será:

`agregar <monto> <categoría> [descripción]` → `listar [--mes AAAA-MM]` → `resumen [--mes AAAA-MM]`

La fecha y el mes de referencia se inyectarán en el núcleo y en la CLI para que los cálculos sean deterministas en pruebas.

## 2. Contexto técnico

- **Lenguaje**: Python 3.12+.
- **Dependencias**: ninguna en ejecución; pytest solo para desarrollo y pruebas.
- **Persistencia**: un archivo JSON local, versionado y escrito de forma atómica.
- **Interfaz**: CLI con `argparse` de la biblioteca estándar.
- **Plataforma**: terminal estándar en Windows, macOS o Linux.
- **Escala**: un usuario, hasta 1000 gastos en el MVP.
- **Restricciones**: sin red, sin concurrencia, sin base de datos y sin interfaz gráfica.
- **Precisión**: montos y totales en centavos enteros; nunca en punto flotante.
- **Rendimiento**: listar o resumir 1000 gastos en menos de 1 segundo en un equipo personal.

## 3. Constitution Check

| Principio | Cumplimiento | Evidencia en este plan |
|---|---|---|
| 1. Stack mínimo | Cumple | Python 3.12+, solo biblioteca estándar y pytest. No hay dependencias de ejecución. |
| 2. La spec manda | Cumple | Cada módulo, decisión y tarea referencia RF-1 a RF-11; la trazabilidad está en la sección 10. |
| 3. Núcleo puro e interfaz fina | Cumple | `gastos/core.py` no importa CLI, almacenamiento ni JSON; `storage.py` y `cli.py` dependen del núcleo. |
| 4. Tests obligatorios | Cumple | Estrategia test-first por RF en la sección 9; toda la suite se ejecutará con `pytest -q`. |
| 5. Persistencia explícita | Cumple | Esquema JSON versionado en la sección 5, validación estricta y escritura atómica con `os.replace`. |
| 6. Idioma consistente | Cumple | Identificadores, módulos y funciones en inglés; comandos, mensajes y documentación de usuario en español. |

## 4. Estructura de módulos (RF-1 a RF-11)

```text
gastos/
├── __init__.py       # Paquete; sin lógica de negocio
├── __main__.py       # Entry point: sys.exit(main())
├── core.py           # Dominio puro: Expense, montos, categorías, meses y resumen (RF-1..RF-8)
├── storage.py        # Carga/guardado JSON, validación y escritura atómica (RF-9, RF-10)
└── cli.py            # argparse, mensajes, salidas y códigos de salida (RF-1..RF-11)

tests/
├── test_core_expense.py   # RF-1, RF-2, RF-3, RF-4
├── test_core_month.py     # RF-5, RF-6
├── test_core_summary.py   # RF-7, RF-8
├── test_storage.py        # RF-9, RF-10
└── test_cli.py            # RF-1..RF-11 (integración)
```

**Decisión de estructura**: un único proyecto Python con paquete `gastos/` y pruebas en `tests/`. No se añade una capa de servicios: dominio puro, almacenamiento y CLI son suficientes para el MVP.

## 5. Modelo de datos JSON (RF-9, RF-10)

### 5.1 Esquema

```json
{
  "version": 1,
  "expenses": [
    {
      "date": "2026-10-26",
      "amount_cents": 1250,
      "category": "Comida",
      "description": "Menú del día"
    },
    {
      "date": "2026-10-27",
      "amount_cents": 320,
      "category": "transporte",
      "description": ""
    }
  ]
}
```

- `version`: entero; el MVP acepta únicamente `1`.
- `expenses`: lista de gastos en orden de registro.
- `date`: fecha local del gasto en formato ISO `YYYY-MM-DD`.
- `amount_cents`: entero positivo; representa centavos de la moneda única.
- `category`: categoría visible, recortada, no vacía y con la grafía original.
- `description`: descripción recortada; cadena vacía si no se indicó.
- No hay `id` porque el MVP no permite editar ni eliminar gastos. El orden del arreglo es el orden de registro.
- El archivo por defecto será `~/.gastos-cli/expenses.json`. La variable de entorno `GASTOS_CLI_DATA_FILE` permitirá usar otra ruta en pruebas y operación local.
- El directorio padre se creará al guardar. La lectura de un archivo ausente devolverá una colección vacía.

### 5.2 Validación y escritura

- Se rechazan JSON inválido, raíz no objeto, `version` distinta de `1`, `expenses` no lista, fechas no ISO, `amount_cents` no entero o no positivo, categorías vacías y descripciones de tipo distinto de cadena.
- La lectura valida todo el archivo antes de devolver datos; si falla, se lanza `DataCorruptionError` y no se escribe nada.
- El guardado escribe en un archivo temporal en el mismo directorio, fuerza la escritura y reemplaza el destino con `os.replace`. Si falla el guardado, el archivo previo queda intacto.
- No hay bloqueo de archivo ni fusión de escrituras concurrentes; el uso concurrente está fuera del MVP.
- Los campos desconocidos se ignoran al leer para no romper datos futuros, pero los campos obligatorios deben existir y tener el tipo correcto.

## 6. Algoritmo de resumen mensual (RF-7, RF-8)

```text
función filter_month(expenses, year, month):
    resultado ← lista vacía
    para cada gasto en expenses:
        si año(gasto.date) = year y mes(gasto.date) = month:
            agregar gasto a resultado
    retornar resultado

función summarize(expenses, year, month):
    filtrados ← filter_month(expenses, year, month)
    total_cents ← 0
    grupos ← diccionario vacío

    para cada gasto en filtrados:
        total_cents ← total_cents + gasto.amount_cents
        clave ← normalizar_categoria(gasto.category)
        si clave no existe en grupos:
            grupos[clave] ← {display: gasto.category, total_cents: 0}
        grupos[clave].total_cents ← grupos[clave].total_cents + gasto.amount_cents

    categorias ← ordenar grupos por:
        1. total_cents descendente
        2. display alfabético ascendente, sin distinguir mayúsculas

    retornar {total_cents, categorias}
```

- `normalizar_categoria(texto)` aplica `strip().casefold()`.
- La categoría mostrada en el resumen es la grafía original de la primera aparición del mes.
- Los totales se suman como enteros de centavos; el formateo a `12.50 €` se hace solo al imprimir.
- La función recibe el año y el mes; no llama a la fecha del sistema para poder probarla de forma determinista.

## 7. Contrato de la CLI (RF-1 a RF-11)

### 7.1 Comandos

```text
python -m gastos agregar "<monto>" "<categoría>" ["<descripción>"]
python -m gastos listar [--mes AAAA-MM]
python -m gastos resumen [--mes AAAA-MM]
python -m gastos --help
```

Los comandos y las opciones visibles están en español porque forman parte de la interfaz de usuario. Las funciones y variables internas serán identificadores en inglés.

### 7.2 Salidas y códigos

| Operación | Resultado esperado | Salida | Código |
|---|---|---|---|
| `agregar` válido | Gasto registrado | stdout: `Gasto registrado: 12.50 € — Comida — 2026-10-26` | 0 |
| `agregar` con monto inválido | Rechazo de dominio | stderr: `Error: el monto debe ser un número positivo con hasta 2 decimales.` | 1 |
| `agregar` con categoría vacía | Rechazo de dominio | stderr: `Error: la categoría no puede estar vacía.` | 1 |
| `listar` con gastos | Listado | stdout: una línea por gasto: `2026-10-26  12.50 €  Comida  Menú del día` | 0 |
| `listar` sin gastos | Listado vacío | stdout: `No hay gastos registrados para 2026-10.` | 0 |
| `resumen` con gastos | Resumen | stdout: `Total: 15.70 €` y líneas `Comida: 12.50 €` | 0 |
| `resumen` sin gastos | Resumen vacío | stdout: `No hay gastos registrados para 2026-10.` y `Total: 0.00 €` | 0 |
| `--mes` inválido | Error de uso | stderr: mensaje de formato y ayuda | 2 |
| datos ilegibles/corruptos | Error de datos | stderr: `Error: no se pudieron leer los datos.` | 1 |
| error de guardado | Error de datos | stderr: `Error: no se pudieron guardar los datos.` | 1 |
| comando/argumentos inválidos | Error de uso | stderr: ayuda de uso en español | 2 |
| `--help` | Ayuda | stdout: ayuda en español | 0 |

Reglas adicionales:

- El monto se acepta con punto decimal y hasta dos decimales; se rechazan signos, comas decimales y separadores de miles.
- El mes por defecto es el mes local actual; el formato explícito es `AAAA-MM` con mes de `01` a `12`.
- La categoría se busca y agrupa con `strip().casefold()`.
- La descripción es opcional; si se omite, se almacena como cadena vacía.
- Los gastos se listan por fecha ascendente y, dentro de la misma fecha, por orden de registro.
- Los errores esperados no muestran traza interna.
- La ruta de datos se inyecta en `main()` para pruebas; por defecto se usa `~/.gastos-cli/expenses.json`.
- La moneda visual es `€`; el MVP no convierte monedas.

## 8. Decisiones técnicas justificadas

| Decisión | Justificación | Alternativa descartada | RF |
|---|---|---|---|
| Python 3.12+ y biblioteca estándar | Cumple el principio 1 y reduce el mantenimiento junior. | Click/Typer | RF-1..RF-11 |
| `argparse` para la CLI | Suficiente para tres comandos, sin dependencias y con ayuda automática. | Parser manual | RF-11 |
| Núcleo puro en `core.py` | Permite probar reglas sin tocar archivos ni terminal. | Lógica dentro de `cli.py` | RF-1..RF-8 |
| JSON local versionado | Legible, inspeccionable y exigido por la constitución. | SQLite | RF-9, RF-10 |
| Montos en centavos enteros | Evita errores de punto flotante y garantiza totales exactos. | `float` | RF-2, RF-7, RF-8 |
| Parseo con `Decimal` y conversión a centavos | Valida dos decimales sin redondeos inesperados. | `float` o redondeo manual | RF-2 |
| Categoría libre normalizada | Permite empezar sin catálogo y agrupar variantes de mayúsculas. | Catálogo fijo | RF-3, RF-7 |
| Sin `id` en el MVP | No hay edición ni eliminación; el orden del arreglo es suficiente. | UUID por gasto | RF-1, RF-9 |
| Fecha local | Coincide con la expectativa del usuario y con “hoy”. | UTC | RF-1, RF-5, RF-7 |
| Escritura atómica | Evita archivos parciales si el proceso falla. | Escritura directa | RF-9, RF-10, NFR-8 |
| Ruta por variable de entorno | Aísla las pruebas sin introducir una opción de usuario adicional. | Ruta fija | RF-9, RF-10 |
| Validación estricta del esquema | Detecta datos corruptos antes de modificarlos. | Lectura permisiva | RF-10 |
| Sin bloqueo de archivo | El uso concurrente está fuera del MVP. | Lock file | Fuera de alcance |
| Pruebas con `main()` y `tmp_path` | Rápidas y deterministas; no requieren subprocesos. | Tests solo por subprocess | RF-1..RF-11 |

## 9. Estrategia de tests

Enfoque test-first: primero se escriben las pruebas de la tarea, se confirma que fallan y después se implementa el código mínimo.

### 9.1 Niveles

- **Unitarias de dominio** (`tests/test_core_*.py`): montos, categorías, descripciones, filtrado por mes y resumen. No usan archivos ni terminal.
- **Unitarias de almacenamiento** (`tests/test_storage.py`): ida y vuelta JSON, archivo ausente, archivo corrupto, validación de esquema y preservación ante fallo de escritura.
- **Integración de CLI** (`tests/test_cli.py`): `main()` con `capsys`, `tmp_path` y fechas fijas; verifican stdout, stderr y códigos de salida.

### 9.2 Matriz de cobertura

| RF | Pruebas previstas |
|---|---|
| RF-1 | Registrar gasto, fecha local, confirmación, recorte y duplicados permitidos. |
| RF-2 | Monto válido, cero, negativo, texto, tres decimales, coma y separador de miles. |
| RF-3 | Categoría vacía, recorte, grafía original y agrupación por mayúsculas/espacios. |
| RF-4 | Descripción omitida, vacía, con espacios exteriores y con contenido especial. |
| RF-5 | Listado del mes actual, orden, formato y mes vacío. |
| RF-6 | Mes explícito, formato inválido y mes sin gastos. |
| RF-7 | Total general, subtotales, orden de categorías, empate y precisión al centavo. |
| RF-8 | Resumen de mes explícito y formato inválido. |
| RF-9 | Registrar, recargar desde otro `main()` y conservar datos. |
| RF-10 | Archivo ausente, JSON inválido, esquema inválido y archivo no sobrescrito. |
| RF-11 | Comando desconocido, argumentos faltantes/sobrantes, errores de dominio, códigos 0/1/2 y ayuda. |

### 9.3 Comandos

```text
pytest -q
```

La suite no requiere red, servicios externos ni la ruta de datos real del usuario.

## 10. Trazabilidad RF → módulo → tarea

| RF | Módulo principal | Tarea |
|---|---|---|
| RF-1 | `core.py`, `cli.py` | T2, T8 |
| RF-2 | `core.py`, `cli.py` | T2, T8 |
| RF-3 | `core.py`, `cli.py` | T2, T8 |
| RF-4 | `core.py`, `cli.py` | T2, T8 |
| RF-5 | `core.py`, `cli.py` | T3, T9 |
| RF-6 | `core.py`, `cli.py` | T4, T9 |
| RF-7 | `core.py`, `cli.py` | T5, T10 |
| RF-8 | `core.py`, `cli.py` | T6, T10 |
| RF-9 | `storage.py`, `cli.py` | T7, T8, T9, T10 |
| RF-10 | `storage.py`, `cli.py` | T7, T8, T9, T10 |
| RF-11 | `cli.py`, `__main__.py` | T8, T9, T10, T11 |

## 11. Riesgos y supuestos

- La moneda `€` y el formato exacto de listado/resumen son supuestos hasta resolver las dudas abiertas de la spec.
- Las categorías son texto libre; no hay catálogo, validación semántica ni límite de longitud.
- El monto se acepta solo con punto decimal; la coma decimal queda rechazada en el MVP.
- No se soportan escrituras concurrentes ni migraciones de esquema.
- Un fallo inesperado del programa no se oculta: solo se capturan los errores esperados de dominio, datos y uso.
- Los campos desconocidos del JSON se ignoran; los campos obligatorios se validan de forma estricta.
