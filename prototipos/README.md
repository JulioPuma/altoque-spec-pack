# Altoque — Prototipos visuales

Prototipos HTML/CSS/JS interactivos generados a partir de [09-ui-spec.md](../speck-pack/09-ui-spec.md) para validar flujos con stakeholders. **No son código de producción.**

## Convención de nombres

`P-XXX-descripcion-corta.html`

- `P-XXX`: ID de la pantalla en el inventario de `09-ui-spec.md` §1.
- `descripcion-corta`: nombre de la pantalla en minúsculas, sin tildes y separado por guiones.

## Cómo usarlos

- Cada archivo se abre directamente en el navegador; no requiere instalación. Abre la carpeta completa para que funcionen los enlaces entre pantallas.
- La **banda punteada superior** no forma parte de la pantalla: permite saltar a cualquier estado y forzar respuestas del servidor (error de red, conflicto, sesión vencida, etc.).
- La sección **Notas para validación** al pie lista el comportamiento a revisar y las reglas de negocio (RN) aplicadas.
- Parámetros de URL útiles: `?estado=` (P-003, P-005, P-006, P-007), `?oportunidad=` (P-004), `?intento=` (P-009, P-011), `?operacion=` (P-013) y `?rol=operador|auditor|desembolso` (P-010 a P-013).

## Índice

### Área del cliente

| ID | Archivo | Descripción | Estados incluidos |
|---|---|---|---|
| P-001 | [P-001-inicio-de-sesion.html](P-001-inicio-de-sesion.html) | Inicio de sesión con cuenta sintética | Inicial, con datos, validación local, enviando, credenciales no válidas, acceso pausado, servicio no disponible, sesión vencida, ingreso correcto |
| P-002 | [P-002-oportunidades.html](P-002-oportunidades.html) | Ofertas, solicitud regular y línea compartida | Cargando, con datos, sin ofertas vigentes, solicitud activa, operación pendiente, iniciando solicitud, actualización fallida, error de lectura, sesión vencida |
| P-003 | [P-003-estado-de-solicitud-regular.html](P-003-estado-de-solicitud-regular.html) | Estado de preevaluación y acciones válidas | Cargando, validando, habilitada (cancelable / con simulación), esperando autorización, rechazada, denegada, no disponible, vencida, diálogo de cancelación, cancelada, fallo recuperable, fallo definitivo, conflicto concurrente, error de lectura, no encontrada, sesión vencida |
| P-004 | [P-004-configurar-simulacion.html](P-004-configurar-simulacion.html) | Monto y plazo para simular | Cargando, editable vacío, con datos, error local, calculando, exitosa, parámetros rechazados, datos ausentes, oportunidad vencida, saldo cambiado, fallo recuperable, fallo definitivo, error de lectura, no encontrada, sesión vencida |
| P-005 | [P-005-detalle-y-aceptacion-de-simulacion.html](P-005-detalle-y-aceptacion-de-simulacion.html) | Costos, cronograma y aceptación expresa | Vigente (cronograma contraído/expandido), aceptando, aceptada, repetición idempotente, vencida, invalidada, información incompleta, conflicto concurrente, sesión vencida, error de red, cargando, vacío, error de lectura |
| P-006 | [P-006-confirmacion-de-contratacion.html](P-006-confirmacion-de-contratacion.html) | Confirmación que separa aceptar de contratar | Listo, procesando, conflicto de saldo, conflicto de vigencia, intento existente, resultado ambiguo, contratación registrada, sesión vencida, cargando, vacío, error de lectura |
| P-007 | [P-007-resultado-de-contratacion.html](P-007-resultado-de-contratacion.html) | Resultado durable de contrato y desembolso | Exitosa con desembolso solicitado/procesando/exitoso/pendiente de gestión, pendiente, revisión manual, fallo recuperable, fallo definitivo, rechazo, lectura no disponible, cargando, vacío |
| P-008 | [P-008-mis-operaciones.html](P-008-mis-operaciones.html) | Lista de operaciones con filtros y paginación | Con resultados, cargando, página adicional, cargando siguiente, actualizando, filtrado sin resultados, vacío, error de red (sin/con lectura previa, al cargar más), sesión vencida |
| P-009 | [P-009-detalle-de-operacion.html](P-009-detalle-de-operacion.html) | Contratación y desembolso, por separado | Contratación pendiente/revisión/recuperable/rechazada/fallida; desembolso solicitado/procesando/pendiente de gestión/consultando/revisión/exitoso/rechazado/fallido; contradicción escalada, actualizando, lectura fallida, no encontrada, sesión vencida |

### Área interna

| ID | Archivo | Descripción | Estados incluidos |
|---|---|---|---|
| P-010 | [P-010-revision-contractual-bandeja.html](P-010-revision-contractual-bandeja.html) | Bandeja de casos contractuales en revisión | Lista, cargando, vacío, filtros sin resultados, error de lectura, lista actualizada, todo cargado, acceso denegado |
| P-011 | [P-011-revision-contractual-caso.html](P-011-revision-contractual-caso.html) | Evidencia, observaciones y resolución contractual | Lectura, cargando, evidencia insuficiente, sin observaciones, editando/validando observación, resolución habilitada (conjunto atómico / rollback), enviando, conflicto 409, resuelto, error de lectura, acceso denegado |
| P-012 | [P-012-revision-de-desembolso-bandeja.html](P-012-revision-de-desembolso-bandeja.html) | Bandeja de desembolsos en revisión manual | Lista, cargando, vacío, sin resultados, error al cargar, error al actualizar, lista actualizada, acceso denegado |
| P-013 | [P-013-revision-de-desembolso-caso.html](P-013-revision-de-desembolso-caso.html) | Evidencia y reconsulta idempotente del desembolso | Lectura, consulta permitida, motivo obligatorio, diálogo de confirmación, consulta en curso, evidencia inconclusa, terminal solo lectura, conflicto 409, editando/registrada observación, cargando, error al cargar, no actualizado, acceso denegado |

## Datos ficticios compartidos

Todas las pantallas usan el mismo caso sintético: clienta Lucía Ramos Huamán (DNI ···1238), línea de S/ 15 000 con S/ 12 000 disponibles, ofertas OFR-7F3A2C / OFR-91B4E0 / OFR-3C1D77 (vencida), solicitud SOL-4E8B19, simulación SIM-A1C9F2 (S/ 5 000 a 12 meses, cuota S/ 459,28, pago S/ 469,28, TCEA 25,03 %, costo total S/ 5 631,44), aceptación ACE-2B7D40, intento CTR-ATT-5C21, contrato CTR-8842 y desembolso DSB-6A90F1. "Hoy" es 14 nov 2026, 13:30 (Lima) en el área del cliente y 14:05 en el área interna.

## Pendiente

- P-014 a P-016 (Could) no tienen prototipo todavía.
