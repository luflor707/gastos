# Constitution — gastos-cli

1. Stack mínimo: Python 3.12+, solo biblioteca estándar y pytest para tests; toda dependencia nueva exige enmienda.
2. La spec manda: ningún comportamiento se implementa sin un RF en `specs/`; si cambia, primero se actualiza la spec.
3. Núcleo puro e interfaz fina: `gastos/core.py` no importa CLI, E/S ni JSON; la CLI no contiene reglas de negocio.
4. Tests obligatorios: cada RF tiene al menos un test automatizado y `pytest -q` debe pasar antes de integrar.
5. Persistencia explícita: JSON local con esquema documentado y escritura atómica; los datos corruptos nunca se sobrescriben sin avisar.
6. Idioma consistente: identificadores y código en inglés; mensajes de usuario, specs y documentación en español.
