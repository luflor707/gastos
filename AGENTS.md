# AGENTS.md — gastos-cli

## Proyecto
CLI en Python para registrar gastos personales y consultar un resumen mensual
por categoría. Núcleo puro (`gastos/core.py`) + almacenamiento
(`gastos/storage.py`) + interfaz CLI (`gastos/cli.py`).

## Comandos
- Ejecutar: `python -m gastos <comando>`
- Tests: `pytest -q`

## Estilo
- Python 3.12+, type hints en todas las funciones públicas.
- Solo biblioteca estándar (pytest únicamente para tests).
- Identificadores en inglés; mensajes de usuario en español.

## Reglas
- Leer `docs/constitution.md` y la spec activa en `specs/` antes de tocar código.
- Ningún comportamiento se implementa sin un RF en la spec.
- No añadir dependencias ni cambiar el formato del JSON sin actualizar la spec.
- No modificar archivos dentro de `specs/` salvo petición explícita.

## Al terminar cualquier tarea
- Ejecutar `pytest -q` y confirmar que todo pasa.
