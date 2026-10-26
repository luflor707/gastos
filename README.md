# gastos-cli

CLI educativa para registrar gastos personales, listarlos por mes y consultar un resumen mensual por categoría. El proyecto está pensado para que una persona desarrolladora junior pueda mantenerlo.

## Estado

**Solo SDD.** La constitución, la especificación, el plan técnico y las tareas están redactados. No hay código, tests implementados ni ejecución de `pytest`.

- [x] T1 — Constitución, spec, plan y tasks.
- [ ] T2 a T12 — Implementación pendiente.

## Objetivo del MVP

Cubrir el ciclo mínimo de gastos personales con tres comandos:

```text
python -m gastos agregar "<monto>" "<categoría>" ["<descripción>"]
python -m gastos listar [--mes AAAA-MM]
python -m gastos resumen [--mes AAAA-MM]
```

Ejemplos:

```text
python -m gastos agregar 12.50 Comida "Menú del día"
python -m gastos listar --mes 2026-10
python -m gastos resumen
```

## Estructura del repositorio

```text
docs/
└── constitution.md

specs/001-gastos-mvp/
├── spec.md
├── plan.md
└── tasks.md

README.md
```

## Documentos SDD

| Documento | Contenido |
|---|---|
| [`docs/constitution.md`](docs/constitution.md) | 6 principios innegociables del proyecto. |
| [`specs/001-gastos-mvp/spec.md`](specs/001-gastos-mvp/spec.md) | Contexto, usuarios, historias, RF-1 a RF-11 con EARS, NFR, casos límite, fuera de alcance, criterios de finalización y dudas abiertas. |
| [`specs/001-gastos-mvp/plan.md`](specs/001-gastos-mvp/plan.md) | Módulos, modelo JSON, algoritmo de resumen, contrato de CLI, decisiones técnicas, estrategia de tests y trazabilidad. |
| [`specs/001-gastos-mvp/tasks.md`](specs/001-gastos-mvp/tasks.md) | Tareas T1 a T12 con dependencias, RF cubiertos y criterio “Hecho cuando”. |

## Requisitos funcionales del MVP

| RF | Resumen |
|---|---|
| RF-1 | Registrar un gasto con fecha local. |
| RF-2 | Validar monto positivo con hasta dos decimales. |
| RF-3 | Validar y agrupar categorías ignorando mayúsculas y espacios exteriores. |
| RF-4 | Tratar la descripción opcional. |
| RF-5 | Listar los gastos del mes actual. |
| RF-6 | Listar los gastos de un mes indicado. |
| RF-7 | Resumir el mes actual con totales por categoría. |
| RF-8 | Resumir un mes indicado. |
| RF-9 | Persistir los gastos entre ejecuciones. |
| RF-10 | Tratar datos ausentes o corruptos sin sobrescribirlos. |
| RF-11 | Gestionar errores, ayuda y códigos de salida. |

## Decisiones clave

- Python 3.12+ y solo biblioteca estándar; pytest únicamente para pruebas.
- Persistencia local en JSON versionado con escritura atómica.
- Montos en centavos enteros para evitar errores de punto flotante.
- Una sola moneda local; el MVP no convierte monedas.
- Categorías de texto libre normalizadas para agrupar.
- Fecha y mes inyectables para pruebas deterministas.
- Separación entre dominio puro, almacenamiento y CLI.

## Dudas abiertas

1. Moneda y símbolo: se propone `€`.
2. Categorías: se propone texto libre normalizado, sin catálogo fijo.
3. Formato exacto del listado y del resumen.

## Próximo paso

Resolver las dudas abiertas, actualizar la spec si cambia alguna decisión y comenzar por T2 siguiendo el orden de `tasks.md`.
