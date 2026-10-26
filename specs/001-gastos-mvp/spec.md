# Especificación: gastos-cli — MVP de gastos personales

**Feature**: `001-gastos-mvp`

**Estado**: Borrador para revisión

**Fecha**: 2026-10-26

## 1. Contexto y objetivo

`gastos-cli` es una herramienta de terminal para registrar gastos personales y consultar cuánto se ha gastado en un mes, con un desglose por categoría. El objetivo del MVP es cubrir el ciclo mínimo: registrar un gasto, listar los gastos de un mes y obtener su resumen. El proyecto es educativo y debe poder ser mantenido por una persona desarrolladora junior.

El MVP trabaja con una única moneda local y no realiza conversiones.

## 2. Usuarios

- **Usuario personal**: persona que usa la terminal y quiere anotar sus gastos sin depender de una aplicación web.
- **Mantenedor junior**: persona que debe entender, probar y modificar el comportamiento a partir de esta especificación.

## 3. Historias de usuario

### US-1 — Registrar un gasto (P1)

Como usuario, quiero registrar un gasto con monto, categoría y descripción opcional para llevar un control básico de mis gastos.

**Prueba independiente**: registrar un gasto válido y comprobar que queda almacenado con la fecha local y los datos introducidos.

### US-2 — Listar los gastos de un mes (P1)

Como usuario, quiero ver los gastos de un mes concreto para revisar en qué se ha ido el dinero.

**Prueba independiente**: con gastos en varios meses, listar un mes y comprobar que solo aparecen los gastos de ese mes.

### US-3 — Consultar el resumen mensual (P1)

Como usuario, quiero ver el total gastado y el desglose por categoría de un mes para entender mi distribución de gastos.

**Prueba independiente**: con gastos conocidos en varias categorías, comprobar el total y los subtotales por categoría.

### US-4 — Conservar los datos entre ejecuciones (P2)

Como usuario, quiero que mis gastos sigan disponibles cuando vuelva a usar la herramienta.

**Prueba independiente**: registrar gastos, cerrar la ejecución y comprobar que el siguiente listado conserva los datos.

## 4. Requisitos funcionales

### RF-1 — Registrar un gasto

- Cuando el usuario registra un gasto con monto y categoría válidos y una descripción opcional, el sistema deberá almacenarlo con la fecha local de hoy.
- Cuando el registro finalice correctamente, el sistema deberá confirmar la operación con un mensaje en español.
- El sistema deberá conservar la categoría y la descripción con sus espacios exteriores recortados.
- El sistema deberá permitir registrar varios gastos con los mismos datos en la misma fecha.

### RF-2 — Validar el monto

- Si el monto falta, no es numérico, es menor o igual que cero, o tiene más de dos decimales, entonces el sistema deberá rechazar el registro con un mensaje en español.
- El sistema deberá aceptar montos enteros y montos con uno o dos decimales.
- El sistema deberá interpretar el punto como separador decimal y no aceptar separadores de miles.
- El sistema deberá tratar el monto como un valor monetario positivo de una moneda única.
- Si el registro se rechaza, entonces el sistema deberá conservar los datos existentes sin cambios.

### RF-3 — Validar y agrupar la categoría

- Si la categoría está vacía o contiene solo espacios, entonces el sistema deberá rechazar el registro con un mensaje en español.
- Cuando la categoría tenga espacios al inicio o al final, el sistema deberá recortarlos antes de almacenarla.
- El sistema deberá conservar la grafía original de la categoría recortada para mostrarla.
- El sistema deberá agrupar las categorías ignorando mayúsculas y espacios exteriores al calcular resúmenes.
- Si el registro se rechaza, entonces el sistema deberá conservar los datos existentes sin cambios.

### RF-4 — Tratar la descripción opcional

- Cuando el usuario proporcione una descripción, el sistema deberá recortar sus espacios exteriores antes de almacenarla.
- Si la descripción se omite o queda vacía después de recortarla, entonces el sistema deberá almacenar una descripción vacía.
- El sistema deberá conservar el contenido de la descripción sin interpretarlo ni ejecutarlo.
- El sistema deberá permitir descripciones con espacios internos, acentos y signos de puntuación.

### RF-5 — Listar los gastos del mes actual

- Cuando el usuario solicita listar sin indicar un mes, el sistema deberá mostrar los gastos del mes local actual.
- El sistema deberá mostrar la fecha, el monto, la categoría y la descripción de cada gasto.
- Si no hay gastos en el mes, entonces el sistema deberá informar en español y finalizar con éxito.
- El sistema deberá ordenar los gastos por fecha ascendente y, dentro de la misma fecha, por orden de registro.

### RF-6 — Listar los gastos de un mes indicado

- Cuando el usuario indica un mes con formato `AAAA-MM`, el sistema deberá mostrar los gastos de ese mes.
- Si el formato del mes no es válido, entonces el sistema deberá rechazar la operación según RF-11.
- Si no hay gastos en el mes indicado, entonces el sistema deberá informar en español y finalizar con éxito.
- El sistema deberá aplicar el mismo contenido y orden que en RF-5.

### RF-7 — Calcular el resumen del mes actual

- Cuando el usuario solicita el resumen sin indicar un mes, el sistema deberá calcular el total y los subtotales por categoría del mes local actual.
- El sistema deberá mostrar el total general y un desglose por categoría con el total de cada una.
- Si no hay gastos en el mes, entonces el sistema deberá mostrar un total de 0 y un mensaje en español.
- El sistema deberá ordenar las categorías por subtotal descendente y, en caso de empate, por nombre alfabético.
- El sistema deberá calcular los totales exactamente al centavo, sin errores de redondeo acumulados.

### RF-8 — Calcular el resumen de un mes indicado

- Cuando el usuario indica un mes con formato `AAAA-MM`, el sistema deberá calcular el resumen de ese mes.
- Si el formato del mes no es válido, entonces el sistema deberá rechazar la operación según RF-11.
- El sistema deberá aplicar las mismas reglas de contenido, orden y precisión que en RF-7.

### RF-9 — Persistir los gastos

- El sistema deberá conservar los gastos entre ejecuciones.
- Cuando una operación finaliza con éxito, el sistema deberá dejar los cambios disponibles para la siguiente ejecución.
- Cuando una operación falla, el sistema deberá conservar los datos previos sin cambios.
- El sistema deberá mantener una única colección local de gastos por usuario.

### RF-10 — Tratar datos ausentes o corruptos

- Si no existen datos almacenados, entonces el sistema deberá iniciar con una colección vacía de gastos.
- Si los datos almacenados no se pueden leer o no tienen la estructura esperada, entonces el sistema deberá abortar la operación con un mensaje en español.
- Si los datos están corruptos, entonces el sistema deberá abstenerse de sobrescribirlos.
- El sistema deberá informar en español que los datos no pudieron leerse.

### RF-11 — Gestionar errores y códigos de salida

- Si el usuario ejecuta un comando desconocido, entonces el sistema deberá mostrar la ayuda de uso en español y finalizar con un código de error.
- Si faltan argumentos obligatorios o sobran argumentos, entonces el sistema deberá mostrar la ayuda de uso en español y finalizar con un código de error.
- Si una operación de dominio falla, entonces el sistema deberá mostrar un mensaje en español y finalizar con un código distinto de cero.
- Cuando una operación finaliza con éxito, el sistema deberá finalizar con código 0.
- Cuando el usuario solicita ayuda, el sistema deberá mostrar la ayuda en español y finalizar con éxito.
- El sistema deberá escribir los mensajes de error en la salida de error y los resultados en la salida estándar.
- El sistema no deberá mostrar trazas internas ante errores esperados.

## 5. Requisitos no funcionales

- **NFR-1 — Local y offline**: el sistema deberá funcionar sin conexión de red y para un único usuario local.
- **NFR-2 — Durabilidad**: los datos confirmados deberán sobrevivir al cierre y a una nueva ejecución.
- **NFR-3 — Respuesta**: listar o resumir hasta 1000 gastos deberá completarse en menos de 1 segundo en un equipo personal convencional.
- **NFR-4 — Mensajes**: todos los mensajes de usuario deberán estar en español y no deberán exponer detalles internos ante errores esperados.
- **NFR-5 — Reproducibilidad**: el cálculo de meses y totales deberá poder verificarse con una fecha y un mes de referencia fijos.
- **NFR-6 — Terminal**: la herramienta deberá funcionar en una terminal estándar, sin interfaz gráfica.
- **NFR-7 — Verificación**: cada requisito funcional deberá tener al menos una prueba automatizada.
- **NFR-8 — Integridad**: ninguna operación fallida deberá dejar datos parciales ni perder datos previos.
- **NFR-9 — Precisión monetaria**: los montos y totales deberán ser exactos al centavo y no acumular errores de punto flotante.
- **NFR-10 — Privacidad**: los datos deberán permanecer en el almacenamiento local del usuario, sin envío a servicios externos.

## 6. Casos límite

- Monto cero, negativo, no numérico o con tres o más decimales: se rechaza según RF-2.
- Monto con coma decimal o separador de miles: se rechaza según RF-2.
- Categoría vacía o compuesta solo por espacios: se rechaza según RF-3.
- Categorías con distinta capitalización o espacios exteriores: se agrupan juntas según RF-3 y RF-7.
- Descripción omitida o vacía: se almacena vacía según RF-4.
- Descripción con saltos de línea, tabs o signos: se conserva tal cual según RF-4.
- Varios gastos idénticos el mismo día: se permiten según RF-1.
- Mes sin gastos: se informa y se finaliza con éxito según RF-5 a RF-8.
- Mes inválido como `2026-13`, `2026-1`, `13-2026` o texto: se rechaza según RF-11.
- Gastos en el primer y último día del mes: pertenecen a ese mes según RF-5 a RF-8.
- Año bisiesto y mes de febrero: se usan las fechas reales del calendario local.
- Cambio de fecha a medianoche o cambio de horario: se usa la fecha local del momento de la operación.
- Datos ausentes: se empieza con una colección vacía según RF-10.
- Datos ilegibles o con estructura inesperada: se aborta sin sobrescribir según RF-10.
- Dos ejecuciones simultáneas: no está soportado en el MVP; no se garantiza la fusión de cambios.
- Totales con muchas categorías o empates: se ordenan según RF-7.

## 7. Fuera de alcance

- Editar o eliminar gastos.
- Registrar ingresos, presupuestos, deudas o gastos recurrentes.
- Soportar varias monedas o convertir entre ellas.
- Gestionar un catálogo fijo de categorías.
- Historial gráfico, estadísticas avanzadas o comparación entre meses.
- Recordatorios, notificaciones o integraciones externas.
- Exportar, importar o sincronizar datos.
- Multiusuario, cuentas, autenticación o uso concurrente.
- Interfaz gráfica, web o API.
- Deshacer operaciones.
- Localización a idiomas distintos del español.
- Migración de datos entre versiones del formato.

## 8. Criterios de finalización

- [ ] Todos los requisitos RF-1 a RF-11 están implementados y sus criterios de aceptación se verifican.
- [ ] Cada requisito funcional tiene al menos una prueba automatizada.
- [ ] La suite completa pasa con `pytest -q`.
- [ ] Los mensajes de usuario están en español y los identificadores del código en inglés.
- [ ] Los datos sobreviven a una nueva ejecución y un fallo no los sobrescribe.
- [ ] Los totales son exactos al centavo y respetan el orden definido.
- [ ] Los códigos de salida y los mensajes de error se comportan según RF-11.
- [ ] Se cumple la constitución del proyecto.
- [ ] `plan.md` y `tasks.md` están actualizados y trazables a los RF.

## 9. Dudas abiertas

- [NECESITA ACLARACIÓN] Moneda y símbolo: se propone una moneda única con símbolo `€`; falta confirmar.
- [NECESITA ACLARACIÓN] Categorías: se propone texto libre normalizado, sin catálogo fijo; falta confirmar.
- [NECESITA ACLARACIÓN] Formato exacto del listado y del resumen: se propone una línea por gasto y una línea por categoría; falta confirmar.
