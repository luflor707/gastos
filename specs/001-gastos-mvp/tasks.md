# Tareas: gastos-cli — MVP de gastos personales

**Feature**: `001-gastos-mvp`

**Entradas**: [`spec.md`](./spec.md) y [`plan.md`](./plan.md)

**Orden**: dependencias de arriba hacia abajo. Cada tarea está pensada para 20-30 minutos.

## Fase 1 — Preparación

- [x] T1 — Congelar constitución, spec, plan y trazabilidad RF-1..RF-11.
  - RF: todos (trazabilidad).
  - Hecho cuando: existen `docs/constitution.md`, `specs/001-gastos-mvp/spec.md`, `specs/001-gastos-mvp/plan.md` y `specs/001-gastos-mvp/tasks.md`, y la tabla de trazabilidad cubre RF-1 a RF-11.

## Fase 2 — Núcleo de dominio

- [ ] T2 — Implementar modelo de gasto, parseo de monto, validación de categoría y descripción en `gastos/core.py`, con pruebas primero.
  - RF: RF-1, RF-2, RF-3, RF-4.
  - Hecho cuando: `pytest -q` pasa pruebas de monto válido/ inválido, categoría vacía/recortada, descripción opcional y conservación de grafía; `add_expense` no hace E/S.

- [ ] T3 — Implementar filtrado y listado del mes actual en `gastos/core.py`.
  - RF: RF-5.
  - Hecho cuando: `pytest -q` pasa pruebas de mes actual, orden por fecha/registro, formato de datos y mes vacío.

- [ ] T4 — Implementar filtrado y listado de un mes explícito `AAAA-MM`.
  - RF: RF-6.
  - Hecho cuando: `pytest -q` pasa pruebas de mes indicado, mes inválido y mes sin gastos.

- [ ] T5 — Implementar `summarize` y agrupación por categoría del mes actual.
  - RF: RF-7.
  - Hecho cuando: `pytest -q` pasa pruebas de total general, subtotales, orden descendente, empate alfabético y suma exacta en centavos.

- [ ] T6 — Implementar `summarize` para un mes explícito.
  - RF: RF-8.
  - Hecho cuando: `pytest -q` pasa pruebas de resumen mensual explícito y reutiliza las reglas de T5 sin duplicar lógica.

## Fase 3 — Persistencia

- [ ] T7 — Implementar carga/guardado JSON versionado, validación de esquema y escritura atómica en `gastos/storage.py`.
  - RF: RF-9, RF-10.
  - Hecho cuando: `pytest -q` pasa ida y vuelta, archivo ausente, JSON corrupto, esquema inválido y preservación del archivo ante fallo de guardado.

## Fase 4 — CLI

- [ ] T8 — Implementar comando `agregar` en `gastos/cli.py` conectado a `core` y `storage`.
  - RF: RF-1, RF-2, RF-3, RF-4, RF-9, RF-10, RF-11.
  - Hecho cuando: agregar un gasto desde `main()` devuelve 0 y persiste; monto/categoría inválidos devuelven 1 sin cambios; uso inválido devuelve 2.

- [ ] T9 — Implementar comando `listar` en `gastos/cli.py` con `--mes`.
  - RF: RF-5, RF-6, RF-9, RF-10, RF-11.
  - Hecho cuando: el listado muestra fecha, monto, categoría y descripción; respeta el mes y el orden; sin gastos informa y devuelve 0; mes inválido devuelve 2.

- [ ] T10 — Implementar comando `resumen` en `gastos/cli.py` con `--mes`.
  - RF: RF-7, RF-8, RF-9, RF-10, RF-11.
  - Hecho cuando: el resumen muestra total y categorías ordenadas; sin gastos muestra total 0; mes inválido devuelve 2; datos corruptos devuelve 1.

- [ ] T11 — Completar `__main__.py`, ayuda, mensajes de error y códigos de salida.
  - RF: RF-11.
  - Hecho cuando: `python -m gastos --help` devuelve 0; comando/argumentos inválidos devuelve 2; errores esperados van a stderr sin traza.

## Fase 5 — Verificación

- [ ] T12 — Ejecutar escenarios de extremo a extremo y revisar criterios de finalización.
  - RF: RF-1..RF-11.
  - Hecho cuando: `pytest -q` pasa completo; agregar/listar/resumir funcionan en una ruta temporal; los datos sobreviven a una segunda ejecución; la checklist de la spec queda revisada.

## Dependencias

- T2 depende de T1.
- T3 depende de T2.
- T4 depende de T3.
- T5 depende de T2 y T3.
- T6 depende de T5.
- T7 depende de T2.
- T8 depende de T2 y T7.
- T9 depende de T4 y T7.
- T10 depende de T6 y T7.
- T11 depende de T8, T9 y T10.
- T12 depende de T11.
