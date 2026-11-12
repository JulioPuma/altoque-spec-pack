# Altoque — Catálogo de reglas de negocio

**Archivo:** `03-reglas-de-negocio.md`  
**Versión:** 0.1  
**Estado:** Catálogo consolidado con guardas pendientes identificadas  
**Convención EARS:** `DEBE` expresa una obligación y `NO DEBE` una prohibición. Se usa condición de estado («Mientras»), evento («Cuando»), respuesta a condición no deseada («Si») o regla ubicua.

Se conservan los IDs RN-001–RN-041 de v0.1. Las reglas compuestas se desglosan con IDs adicionales y las nuevas reglas citan sus decisiones. Los datos de ejemplo son sintéticos y no fijan valores de configuración.

## 1. Acceso, vigencia y simulación

| ID | Regla EARS | Prioridad | Fuente |
|---|---|---|---|
| RN-001 | Cuando un cliente accede a operaciones crediticias, el sistema DEBE verificar una sesión autenticada válida. | MUST | PRINCIPIO-006 |
| RN-002 | Cuando un cliente consulta una oferta, el sistema NO DEBE mostrar una oferta de otro cliente. | MUST | M-01 |
| RN-003 | Cuando verifica el límite temporal de una oferta preferencial, el sistema DEBE usar el intervalo UTC desde su emisión, inclusive, hasta la emisión más 30 días calendario, exclusive. | MUST | DEC-79, DEC-101 |
| RN-004 | Mientras la oferta aplicable carezca de vigencia, el sistema NO DEBE permitir su contratación. | MUST | Ronda 6 |
| RN-005 | El sistema DEBE calcular condiciones financieras en el único módulo financiero backend. | MUST | PRINCIPIO-003 |
| RN-006 | El frontend NO DEBE mantener fórmulas financieras independientes ni recalcular condiciones financieras. | MUST | PRINCIPIO-003 |
| RN-007 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar el monto. | MUST | PRINCIPIO-004 |
| RN-008 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar el plazo. | MUST | PRINCIPIO-004 |
| RN-009 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar la cuota financiera de cada período. | MUST | DEC-51 |
| RN-010 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar los intereses. | MUST | PRINCIPIO-004 |
| RN-011 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar la TCEA. | MUST | DEC-49 |
| RN-012 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar el cronograma. | MUST | PRINCIPIO-004 |

**CA RN-001/002:** `CLI-001` autenticado consulta `OFR-100` propia: permitido; `CLI-002` consulta la misma oferta: contenido no revelado.  
**CA RN-003/004:** una oferta preferencial emitida puede continuar antes del límite. En el instante de emisión más 30 días calendario UTC se considera vencida.

## 2. Aceptación y contratación

| ID | Regla EARS | Prioridad | Fuente |
|---|---|---|---|
| RN-013 | Cuando cambia cualquier condición financiera, el sistema DEBE exigir nueva simulación. | MUST | Ronda 6 |
| RN-014 | Cuando cambia la versión simulada, el sistema DEBE exigir nueva aceptación expresa. | MUST | Ronda 6 |
| RN-015 | Cuando se inicia una contratación, el sistema DEBE verificar que la aceptación corresponda a la versión íntegra e inmutable de la simulación. | MUST | PRINCIPIO-005 |
| RN-016 | Al registrar un contrato, el sistema DEBE confirmar en la misma transacción el contrato, su auditoría obligatoria, la orden única de desembolso y la conversión de la reserva en consumo. | MUST | INV-04, INV-12 |
| RN-017 | Cuando existe un contrato para la oferta preferencial o la solicitud regular, el sistema NO DEBE registrar un segundo contrato para esa misma oportunidad. | MUST | INV-03, DEC-62 |
| RN-018 | Cuando finaliza un intento de contratación, el sistema DEBE persistir su resultado. | MUST | DEC-60 |

**CA RN-013–015:** si `SIM-002` cambia plazo respecto de `SIM-001`, `ACE-001` no autoriza contratar `SIM-002`.  
**CA RN-016–018:** si falla la auditoría de `CTR-ATT-003`, no se declara contrato exitoso; el intento conserva identidad y resultado.

## 3. Desembolso y recuperación

| ID | Regla EARS | Prioridad | Fuente |
|---|---|---|---|
| RN-019 | Cuando se recibe un segundo comando de ejecución de desembolso para un contrato, el sistema DEBE rechazarlo por duplicidad. | MUST | Ronda 6 |
| RN-020 | Cuando el core confirma éxito inequívoco, el sistema DEBE registrar `DESEMBOLSO_SIMULADO_EXITOSO`. | MUST | Rondas 6–7 |
| RN-021 | Cuando el resultado sea ambiguo o expire la espera, el sistema NO DEBE declarar éxito ni fallo. | MUST | INV-06 |
| RN-022 | Cuando el resultado sea ambiguo, el sistema DEBE registrar `DESEMBOLSO_PENDIENTE_DE_GESTION`. | MUST | Ronda 6 |
| RN-023 | Cuando se recupera un desembolso pendiente, el sistema DEBE consultar la operación original. | MUST | Ronda 7 |
| RN-024 | Durante la recuperación, el sistema NO DEBE crear otro identificador ni iniciar una segunda operación lógica de desembolso; solo puede consultar o reentregar idempotentemente el comando original. | MUST | INV-05, INV-14 |
| RN-025 | Cuando la recuperación automática no resuelva el resultado, el sistema DEBE escalarlo a revisión manual. | MUST | DEC-76, PA-12 |

**CA RN-019–025:** ante timeout de `DSB-009`, una repetición no ejecuta otra transferencia; se consulta `DSB-009` y se conserva pendiente si no hay evidencia.

## 4. KPIs, accesibilidad y estados

| ID | Regla EARS | Prioridad | Fuente |
|---|---|---|---|
| RN-026 | Cuando una oferta se visualiza varias veces por el mismo cliente, el sistema DEBE contarla una vez para el KPI. | MUST | Ronda 5 |
| RN-027 | Cuando se atribuye una conversión, el sistema DEBE asignarla a la semana de contratación. | MUST | Ronda 6 |
| RN-028 | El sistema DEBE permitir operar mediante teclado los flujos principales. | MUST | PRINCIPIO-012 |
| RN-029 | Cuando se confirma una aceptación válida, el sistema DEBE conservarla vinculada a su propia simulación. | MUST | Ronda de refinamiento 5, sustituye la competencia entre simulaciones de DEC-55 y DEC-64 |
| RN-030 | Mientras la contratación posea el bloqueo transaccional por oportunidad, el sistema DEBE hacer esperar las nuevas aceptaciones de esa oportunidad y revalidarlas antes de continuar. | MUST | INV-15 |
| RN-031 | Cuando se solicita recuperar un estado de fallo, el sistema DEBE exigir clasificación recuperable y un reintento controlado. En contratación, solo `CONTRATACION_FALLIDA_RECUPERABLE` admite esta recuperación. | MUST | DEC-59, DEC-96 |
| RN-032 | Cuando un estado terminal está confirmado, el sistema NO DEBE modificarlo. | MUST | DEC-61 |
| RN-033 | Cuando se requiere corregir un estado terminal, el sistema DEBE crear una nueva operación o registro de ajuste. | MUST | DEC-61 |
| RN-034 | Cuando se crea un intento de contratación, el sistema DEBE asignarle identidad y estado propios. | MUST | DEC-60 |
| RN-035 | Cuando se registra un contrato, el sistema DEBE mantenerlo único frente a todos los intentos relacionados. | MUST | DEC-60 |

## 5. Reglas financieras añadidas

| ID | Regla EARS | Prioridad | Fuente |
|---|---|---|---|
| RN-036 | Cuando se calcula la tasa mensual, el sistema DEBE resolver `i_m=(1+TEA)^(1/12)-1` con precisión interna de 34 dígitos y redondear el resultado a 16 decimales con `HALF_UP`. | MUST | DEC-49 / Ronda 8, PA-07 |
| RN-037 | Cuando se calcula la cuota financiera, el sistema DEBE usar el sistema francés. | MUST | Ronda 8 consolidada |
| RN-038 | Cuando se calcula un pago, el sistema DEBE sumar el seguro fijo global a la cuota financiera. | MUST | DEC-51 |
| RN-039 | Cuando se redondea un importe monetario, el sistema DEBE aplicar `HALF_UP` a dos decimales. | MUST | DEC-50, DEC-84, PA-07 |
| RN-040 | Cuando se calcula la última cuota, el sistema DEBE ajustarla para cancelar exactamente el saldo. | MUST | DEC-50 |
| RN-041 | Cuando se calcula la TCEA, el sistema DEBE usar el flujo de desembolso y las cuotas con seguro. | MUST | DEC-49 |

## 6. Reglas de consolidación v0.1

| ID | Regla EARS | Prioridad | Fuente |
| --- | --- | --- | --- |
| RN-042 | Cuando el sistema identifica un desembolso, DEBE usar un identificador único y persistente asociado al contrato. | MUST | DEC-63 |
| RN-043 | Cuando se repite una petición con la misma clave y datos, el sistema DEBE consultar o devolver el resultado de la operación original. | MUST | DEC-66 |
| RN-044 | Cuando se recibe una clave nueva, el sistema DEBE validar si la operación puede ejecutarse. | MUST | DEC-66 |
| RN-045 | Cuando se consulta una contratación cuya respuesta se perdió después del registro, el sistema DEBE devolver el contrato existente al cliente autorizado. | MUST | DEC-67 |
| RN-046 | Si falla la auditoría obligatoria durante el registro contractual, el sistema DEBE revertir la transacción. | MUST | DEC-68 |
| RN-047 | Cuando se confirma la reversión por fallo de auditoría, el sistema DEBE registrar el fallo del intento por separado. | MUST | DEC-68 |
| RN-048 | Cuando se solicita otro intento de contratación, el sistema DEBE comprobar que no exista contrato ni intento previo con resultado incierto antes de permitirlo. | MUST | DEC-69 |
| RN-049 | Si el resultado de la contratación es incierto, el sistema DEBE mantener el intento en CONTRATACION_PENDIENTE. | MUST | DEC-70 |
| RN-050 | Mientras la contratación esté pendiente, el sistema DEBE consultar evidencia antes de resolverla. | MUST | DEC-70 |
| RN-051 | Si el core confirma el desembolso y falla la persistencia local, el sistema DEBE tratar el resultado como pendiente y reconciliarlo consultando el core. | MUST | DEC-73 |
| RN-052 | Si el core responde «operación no encontrada», el sistema DEBE mantener el desembolso pendiente hasta obtener evidencia concluyente. | MUST | DEC-74 |
| RN-053 | Cuando se confirma DESEMBOLSO_FALLIDO definitivo, el sistema DEBE disponer de evidencia concluyente de que el desembolso no se ejecutó y no puede completarse. | MUST | DEC-75 |
| RN-054 | Cuando dos procesos reconcilian el mismo desembolso concurrentemente, el sistema DEBE confirmar como máximo una transición desde el mismo estado y versión. | MUST | DEC-77 |
| RN-055 | Mientras se recupera un desembolso, el sistema DEBE limitar los reintentos técnicos a consulta, reentrega idempotente del mismo comando o persistencia del resultado original. | MUST | DEC-78, INV-14 |
| RN-056 | Si la oferta vence tras la aceptación y antes del registro, el sistema DEBE exigir una oferta o recorrido vigente para contratar. | MUST | DEC-80 |
| RN-057 | Cuando se comprueba la vigencia de una simulación exitosa, el sistema DEBE considerarla vencida desde el instante de creación más 30 minutos. | MUST | DEC-81, DEC-101 |
| RN-058 | Cuando se genera una nueva simulación, el sistema DEBE conservar las simulaciones anteriores. | MUST | DEC-82 |
| RN-059 | Cuando se genera otra simulación, el sistema NO DEBE invalidar por ese solo hecho una aceptación asociada a otra simulación. | MUST | Ronda de refinamiento 3, sustituye DEC-82 para esta regla |
| RN-060 | El sistema DEBE validar los parámetros financieros contra los límites configurables y valores iniciales definidos en RN-195–202. | MUST | DEC-83, PA-08 |
| RN-061 | Cuando se construye el cronograma, el sistema DEBE programar cuotas mensuales desde el mes siguiente con ajuste de fin de mes. | MUST | DEC-85, DEC-100 |
| RN-062 | Cuando la TEA es cero, el sistema DEBE calcular la cuota financiera como P/n. | MUST | DEC-86 |
| RN-063 | Si no existe una TCEA válida, el sistema DEBE registrar la simulación como `SIMULACION_FALLIDA_DEFINITIVA`. | MUST | DEC-86, PA-22 |
| RN-064 | Si no existe una TCEA válida, el sistema DEBE mostrar un error explícito. | MUST | DEC-86 |
| RN-065 | Cuando un actor se autentica, el sistema DEBE asignarle uno de los roles funcionales aprobados y limitarlo a las acciones y recursos autorizados para ese rol. El mecanismo técnico se define en la especificación de seguridad. | MUST | DEC-87, PA-15 |
| RN-066 | Cuando ocurre un error técnico, el sistema DEBE clasificarlo por operación como recuperable, definitivo o incierto y conservar la causa de la clasificación. | MUST | DEC-90, PA-03 |
| RN-067 | Si una clave idempotente se reutiliza con datos distintos, el sistema DEBE rechazar la petición por conflicto sin ejecutarla. | MUST | DEC-91 |
| RN-068 | El sistema DEBE conservar el registro de idempotencia mientras sea necesario para garantizar la unicidad, recuperar el resultado original y auditar la operación; si existe contrato, DEBE conservarlo durante la vigencia del registro contractual. | MUST | DEC-92, PA-29 |
| RN-069 | Cuando el sistema representa un estado en persistencia, eventos o API, DEBE usar el nombre canónico prefijado de su máquina. | MUST | DEC-95 |
| RN-070 | Cuando un fallo se confirma como definitivo, el sistema DEBE contar con evidencia que lo sustente antes de tratar el estado como terminal. | MUST | DEC-96 |
| RN-071 | Cuando se procesa una aceptación, simulación o contratación, el sistema DEBE serializarla por oportunidad de crédito y aplicar el orden de bloqueo de INV-15 cuando también requiera la línea. | MUST | INV-15 |
| RN-072 | Mientras la revisión manual carezca de evidencia concluyente, el sistema DEBE conservar pendiente el resultado del desembolso. | MUST | DEC-98 |
| RN-073 | Cuando se realizan cálculos financieros, el sistema DEBE usar `BigDecimal`, precisión interna de 34 dígitos, `HALF_UP` y períodos financieros mensuales uniformes. | MUST | DEC-100, PA-07 |
| RN-074 | Cuando se verifica un vencimiento, el sistema DEBE comparar instantes UTC y considerar vencido el recurso en el instante límite. | MUST | DEC-101 |
| RN-075 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar el seguro de cada cuota. | MUST | DEC-51 |
| RN-076 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar el pago total de cada cuota. | MUST | DEC-51 |
| RN-077 | Cuando una simulación finaliza correctamente, el sistema DEBE mostrar el costo total. | MUST | DEC-49 |
| RN-078 | Cuando se valida una contratación, el sistema DEBE comprobar la vigencia de la oferta aplicable, la simulación y la aceptación asociada. | MUST | Ronda de refinamiento 5 |
| RN-079 | Cuando finaliza un intento de contratación, el sistema DEBE conservar su correlación. | MUST | DEC-60, DEC-71 |
| RN-080 | Cuando se presenta un control de un flujo principal, el sistema DEBE proporcionar una etiqueta identificable. | MUST | DEC-93 |
| RN-081 | Cuando se confirma una transición, el sistema DEBE auditar los campos de INV-09. | MUST | DEC-71, DEC-94 |
| RN-082 | Cuando se intenta una transición, el sistema DEBE validar estado origen y versión de entidad antes de confirmarla. | MUST | DEC-72 |
| RN-083 | Si la simulación está incompleta o es inválida, el sistema NO DEBE publicarla como SIMULACION_EXITOSA. | MUST | DEC-86 |
| RN-084 | Cuando se calcula el costo total, el sistema DEBE sumar los pagos del cronograma con seguro incluido. | MUST | DEC-51 |
| RN-085 | En la comprobación final del registro contractual, después de adquirir los bloqueos de oportunidad y línea y justo antes de escribir el conjunto contractual, el sistema DEBE verificar con el reloj UTC autoritativo de la base de datos que la oferta aplicable, la simulación y la aceptación asociada sigan vigentes. | MUST | DEC-80, Ronda de refinamiento 5, PA-23 |
| RN-086 | Cuando el sistema valida el crédito disponible para contratar, DEBE usar la línea única del cliente, compartida por los recorridos preferencial y regular. | MUST | Ronda de refinamiento 4 |
| RN-087 | Cuando se crea un nuevo intento permitido, el sistema DEBE confirmar atómicamente la identidad del intento y la reserva del monto solicitado respecto del saldo disponible de la línea. | MUST | INV-11 |
| RN-088 | Si el monto solicitado supera el saldo disponible al intentar la reserva, el sistema NO DEBE continuar el registro contractual. | MUST | Ronda de refinamiento 4 |
| RN-089 | Mientras el resultado de la contratación sea incierto, el sistema DEBE mantener la reserva del intento original. | MUST | Ronda de refinamiento 4 |
| RN-090 | Cuando se confirma el contrato, el sistema DEBE convertir la reserva en consumo de la línea dentro de la misma transacción del contrato, la auditoría obligatoria y la orden única de desembolso, sin duplicar el descuento. | MUST | INV-12 |
| RN-091 | Cuando el intento alcanza `CONTRATACION_RECHAZADA` o `CONTRATACION_FALLIDA_DEFINITIVA`, el sistema DEBE liberar su reserva exactamente una vez. | MUST | Ronda de refinamiento 4, PA-36 |
| RN-092 | Cuando el mismo cliente repite una aceptación para la misma simulación y versión inmutable, el sistema DEBE devolver la aceptación existente sin duplicar su registro. | MUST | INV-16 |
| RN-093 | Cuando la preevaluación habilita una oferta base, el sistema DEBE permitir al cliente simularla conforme a sus guardas. | MUST | Ronda de refinamiento 5 |
| RN-094 | Si la preevaluación exige evaluación adicional, el sistema DEBE informar ese resultado al cliente. | MUST | Ronda de refinamiento 5 |
| RN-095 | Mientras la evaluación adicional exigida carezca de autorización, el sistema NO DEBE permitir la contratación. | MUST | Ronda de refinamiento 5 |
| RN-096 | Si el instante autoritativo de la comprobación final es igual o posterior a cualquier vencimiento exigido por RN-085, el sistema DEBE rechazar el registro contractual. Si es anterior a todos los límites, ese instante constituye el punto de decisión de vigencia de la transacción. | MUST | Ronda de refinamiento 5, PA-23 |
| RN-097 | Cuando la preevaluación habilita una oferta base, el sistema NO DEBE permitir un monto solicitado superior al máximo informado por la preevaluación. | MUST | Ronda de refinamiento 6, PA-08 |
| RN-098 | Cuando calcula una simulación del flujo regular, el sistema DEBE aplicar la TEA pizarra configurada. | MUST | Ronda de refinamiento 6 |
| RN-099 | Cuando registra el contrato, el sistema DEBE confirmar en la misma transacción la orden de desembolso con su identificador persistente y único, conforme al conjunto atómico de RN-016. | MUST | INV-12 |
| RN-100 | Después de una interrupción, el sistema solo DEBE permitir que otro despachador tome la orden cuando esta permanezca durablemente en `DESEMBOLSO_SOLICITADO` y no exista una adquisición confirmada. | MUST | INV-14 |
| RN-101 | Si el envío original pudo haber comenzado, el sistema DEBE consultar su resultado por el identificador persistente y aplicar RN-024 y RN-025. | MUST | Ronda de refinamiento 6 |
| RN-102 | Si se confirma el fallo definitivo del desembolso, el sistema DEBE conservar el contrato registrado. | MUST | Ronda de refinamiento 6 |
| RN-103 | Si se confirma el fallo definitivo del desembolso, el sistema DEBE conservar el consumo de la línea asociado al contrato. | MUST | Ronda de refinamiento 6 |
| RN-104 | Cuando se autorice liberar el consumo tras un fallo definitivo del desembolso, el sistema DEBE registrar la liberación como ajuste auditable. | MUST | Ronda de refinamiento 6, política de autorización pendiente |
| RN-105 | Cuando el cliente autenticado acepta expresamente una simulación vigente, el sistema DEBE conservar la identidad del cliente, la fecha y hora, la acción expresa y la referencia a la versión inmutable que contiene todas las condiciones mostradas. | MUST | Ronda de refinamiento 7 |
| RN-106 | Cuando el cliente selecciona «Contratar», el sistema DEBE usar exactamente la aceptación identificada para esa simulación. | MUST | Ronda de refinamiento 7 |
| RN-107 | Antes de iniciar el registro contractual, el sistema DEBE persistir atómicamente la identidad del intento, su vínculo con la oportunidad y la aceptación, y su reserva. | MUST | INV-11 |
| RN-108 | Mientras falte confirmar si el intento original registró un contrato o terminó sin contrato, el sistema NO DEBE permitir otro intento para la misma oportunidad. | MUST | Ronda de refinamiento 7 |
| RN-109 | Si la preevaluación no responde, el sistema DEBE informar al cliente que el resultado no está disponible. | MUST | Ronda de refinamiento 7 |
| RN-110 | Si la preevaluación no responde, el sistema DEBE permitir volver a consultar su resultado. | MUST | Ronda de refinamiento 7 |
| RN-111 | Si la preevaluación confirma que no habilita una oferta base, el sistema DEBE informar ese resultado al cliente. | MUST | Ronda de refinamiento 7 |
| RN-112 | Si la preevaluación confirma que no habilita una oferta base, el sistema NO DEBE permitir simular ni contratar en el flujo regular correspondiente. | MUST | Ronda de refinamiento 7 |
| RN-113 | Cuando se determina la vigencia de una aceptación, el sistema DEBE usar el primer vencimiento aplicable entre la oferta preferencial o la versión de preevaluación que habilitó la oferta base, su autorización adicional si corresponde y la simulación. | MUST | Ronda de refinamiento 8, PA-23 |
| RN-114 | Si se invalida la versión aceptada, el sistema NO DEBE utilizar su aceptación para contratar. | MUST | Ronda de refinamiento 8 |
| RN-115 | Cuando el mismo cliente repite una petición de contratación con la misma clave y los mismos datos, el sistema DEBE devolver el intento original y su estado actual. | MUST | Ronda de refinamiento 8 |
| RN-116 | Si el mismo cliente reutiliza la clave de contratación con datos distintos, el sistema DEBE aplicar RN-067. | MUST | Ronda de refinamiento 8 |
| RN-117 | Cuando el cliente presenta una nueva clave de contratación, el sistema DEBE aplicar las guardas vigentes de unicidad, intento previo incierto y saldo disponible. | MUST | Ronda de refinamiento 8 |
| RN-118 | Cuando un cliente autenticado consulta una contratación propia, el sistema DEBE mostrar su estado actual, pendiente o resuelto. | MUST | Ronda de refinamiento 8 |
| RN-119 | Cuando un cliente autenticado consulta un desembolso propio, el sistema DEBE mostrar su estado actual, pendiente o resuelto. | MUST | Ronda de refinamiento 8 |
| RN-120 | Cuando el cliente solicita el estado de una contratación o desembolso, el sistema DEBE verificar la titularidad antes de devolver información. | MUST | Ronda de refinamiento 8 |
| RN-121 | Mientras se opera en la primera evaluación cerrada, el sistema DEBE permitir la autenticación únicamente a cuentas sintéticas creadas previamente para la evaluación. | MUST | Decisión de cierre de brechas de acceso |
| RN-122 | Cuando un cliente solicita un recurso de crédito, el sistema DEBE verificar su titularidad antes de devolver información o ejecutar una acción. | MUST | M-01 y Ronda de refinamiento 8 |
| RN-123 | Cuando la respuesta del registro contractual sea incierta, el sistema DEBE recuperar el intento persistido por su identidad y conservar pendiente el resultado hasta obtener evidencia concluyente. | MUST | Rondas de refinamiento 3 y 7 |
| RN-124 | Cuando se configura una cuenta sintética para la primera evaluación, su contraseña DEBE ser independiente de su DNI sintético y NO DEBE ser igual ni derivarse de ese identificador. | MUST | PRINCIPIO-013, M-01 |
| RN-125 | Cuando se recupera una contratación cuya respuesta se perdió, el sistema DEBE consultar el intento y el conjunto contractual mediante una lectura durable y autoritativa. | MUST | INV-13 |
| RN-126 | Cuando la lectura durable confirma contrato, auditoría obligatoria, orden única y consumo, el sistema DEBE resolver el intento como `CONTRATACION_EXITOSA` y devolver el contrato al cliente autorizado. | MUST | INV-12, INV-13 |
| RN-127 | Cuando existe evidencia durable de que la transacción contractual fue revertida o cerró sin contrato, el sistema DEBE clasificar el resultado como rechazo de negocio, fallo técnico recuperable o fallo técnico definitivo y aplicar RN-178–180. | MUST | INV-13, RN-091, PA-36 |
| RN-128 | Mientras la recuperación contractual no confirme ni el conjunto contractual ni el cierre sin contrato, el sistema DEBE conservar `CONTRATACION_PENDIENTE`, mantener la reserva y bloquear otro intento para la misma oportunidad. | MUST | INV-13 |
| RN-129 | Después de confirmar la transacción contractual, el sistema DEBE iniciar el envío al core únicamente desde la orden durable ya registrada. | MUST | INV-12, PRINCIPIO-008 |
| RN-130 | Cuando un despachador toma una orden en `DESEMBOLSO_SOLICITADO`, el sistema DEBE garantizar su adquisición exclusiva mediante bloqueo o leasing durable. | MUST | INV-14 |
| RN-131 | Antes de efectuar la llamada inicial al core, el despachador DEBE confirmar durablemente la transición de `DESEMBOLSO_SOLICITADO` a `DESEMBOLSO_PROCESANDO`. | MUST | INV-14 |
| RN-132 | Cuando un despachador confirma `DESEMBOLSO_PROCESANDO`, solo ese proceso DEBE efectuar la primera entrega asociada a esa adquisición. Las entregas técnicas posteriores pertenecen a la recuperación de la misma operación. | MUST | INV-14 |
| RN-133 | Después de una caída o pérdida de respuesta en `DESEMBOLSO_PROCESANDO`, el sistema DEBE consultar la operación original y solo puede reentregar el comando usando exactamente el mismo identificador persistente y los mismos datos. | MUST | INV-14, RN-023–024 |
| RN-134 | Cuando se agota la política aprobada de consultas y reentregas idempotentes sin resultado concluyente, el sistema DEBE escalar el desembolso a revisión manual sin crear otra operación lógica. | MUST | RN-025, PA-03, PA-12 |
| RN-135 | Cuando el core simulado recibe nuevamente un identificador con los mismos datos, DEBE devolver la operación original sin ejecutar otro desembolso simulado; si los datos son distintos, DEBE rechazar por conflicto sin ejecutar. | MUST | INV-05, INV-14 |
| RN-136 | Cuando una contratación requiere bloquear la oportunidad y la línea, el sistema DEBE adquirir primero la oportunidad y después la línea del cliente. | MUST | INV-15 |
| RN-137 | El sistema NO DEBE ejecutar ningún flujo que adquiera el bloqueo de línea antes del bloqueo de oportunidad. | MUST | INV-15 |
| RN-138 | Después de esperar un bloqueo de oportunidad, el sistema DEBE revalidar el estado y la versión de la oportunidad, la aceptación seleccionada, sus vigencias, la ausencia de contrato y la ausencia de intento incierto. | MUST | INV-15 |
| RN-139 | Después de esperar el bloqueo de línea, el sistema DEBE revalidar el saldo disponible y la versión de la línea antes de crear la reserva. | MUST | INV-15 |
| RN-140 | Cuando una aceptación continúa después de esperar a una contratación, el sistema DEBE rechazarla si ya existe contrato para la oportunidad. | MUST | INV-15 |
| RN-141 | Cuando una aceptación continúa después de que la contratación cierra sin contrato, el sistema DEBE permitirla solo si su simulación y la oportunidad siguen vigentes y se satisfacen las demás guardas. | MUST | INV-15 |
| RN-142 | Cuando contrataciones de oportunidades distintas compiten por la misma línea, el sistema DEBE serializar sus reservas mediante el bloqueo y la versión de esa línea. | MUST | INV-15 |
| RN-143 | Cuando se convierte o libera una reserva, el sistema DEBE validar su estado y versión y confirmar como máximo un efecto para esa transición. | MUST | INV-16 |
| RN-144 | Cuando el core simulado responde `RECIBIDA` o `PROCESANDO`, el sistema DEBE mantener el resultado pendiente y consultar la operación original. | MUST | Cierre de PA-04 |
| RN-145 | Cuando el core simulado responde `EXITOSA`, el sistema DEBE confirmar `DESEMBOLSO_SIMULADO_EXITOSO` solo después de persistir localmente el resultado. | MUST | INV-17 |
| RN-146 | Cuando el core simulado responde `RECHAZADA`, el sistema DEBE confirmar `DESEMBOLSO_RECHAZADO` solo después de persistir localmente el resultado. | MUST | INV-17 |
| RN-147 | Cuando el core simulado responde `FALLIDA_DEFINITIVA`, el sistema DEBE confirmar `DESEMBOLSO_FALLIDO` solo si la respuesta acredita que la operación no se ejecutó ni puede completarse y después de persistir localmente el resultado. | MUST | RN-053, INV-17 |
| RN-148 | Cuando el core simulado responde `NO_ENCONTRADA`, el sistema DEBE tratar la consulta como inconclusa y puede reentregar idempotentemente el comando original conforme a RN-133; NO DEBE confirmar un fallo ni crear otra operación. | MUST | RN-052, INV-14 |
| RN-149 | Cuando ocurre timeout, desconexión, respuesta inválida o código de resultado desconocido, el sistema DEBE tratar el resultado como inconcluso. | MUST | INV-06 |
| RN-150 | Cuando una consulta descubre `RECHAZADA`, el sistema DEBE permitir la transición desde `DESEMBOLSO_CONSULTANDO_ESTADO` o `DESEMBOLSO_EN_REVISION_MANUAL` a `DESEMBOLSO_RECHAZADO`. | MUST | Cierre de PA-04 |
| RN-151 | Cuando llega una respuesta tardía, el sistema DEBE aplicarla únicamente si el estado local no es terminal y la versión esperada sigue vigente. | MUST | INV-10, INV-17 |
| RN-152 | Cuando una respuesta contradice un estado terminal local confirmado, el sistema NO DEBE modificar el terminal y DEBE registrar ambas evidencias y escalar el conflicto a revisión manual. | MUST | INV-07, INV-17 |
| RN-153 | Cuando el core simulado confirma un resultado terminal, DEBE conservarlo inmutable y devolverlo en las consultas posteriores de la misma operación. | MUST | INV-17 |
| RN-154 | Cuando se confirma una preevaluación, el sistema DEBE conservar `resultadoId`, cliente, `solicitudId` regular previamente creado, versión, estado, `creadoEn` y `venceEn`. | MUST | INV-18, PA-38 |
| RN-155 | Cuando se calcula la vigencia de una preevaluación, el sistema DEBE usar el intervalo desde `creadoEn`, inclusive, hasta `creadoEn` más 24 horas, exclusive, medido en UTC. | MUST | INV-18 |
| RN-156 | Cuando la preevaluación devuelve `OFERTA_BASE_HABILITADA`, DEBE incluir un monto máximo positivo y su moneda. | MUST | INV-18 |
| RN-157 | Cuando la preevaluación devuelve `EVALUACION_ADICIONAL_REQUERIDA`, DEBE incluir un `evaluacionId` asociado al cliente y a la solicitud. | MUST | INV-18 |
| RN-158 | Mientras la autorización adicional esté `PENDIENTE`, el sistema DEBE mantener `SOLICITUD_ESPERANDO_AUTORIZACION`, bloquear simulación y contratación y permitir consultar su estado. | MUST | INV-18, PA-38 |
| RN-159 | Cuando la autorización adicional queda `DENEGADA`, el sistema DEBE informar que no existe oferta habilitada y bloquear la simulación y contratación de esa solicitud. | MUST | INV-18 |
| RN-160 | Cuando la autorización adicional queda `AUTORIZADA`, el sistema DEBE permitir continuar solo hasta el vencimiento del resultado de preevaluación asociado. | MUST | INV-18 |
| RN-161 | Cuando la preevaluación devuelve `SIN_OFERTA`, el sistema DEBE bloquear la simulación y contratación de la solicitud correspondiente. | MUST | INV-18 |
| RN-162 | Cuando la preevaluación devuelve `NO_DISPONIBLE`, el sistema DEBE informar indisponibilidad y permitir una reconsulta sin inferir ausencia de oferta. | MUST | INV-18 |
| RN-163 | Cuando una respuesta de preevaluación omite un dato obligatorio para su estado, el sistema DEBE tratarla como `NO_DISPONIBLE` y NO DEBE completar el dato por defecto. | MUST | INV-18 |
| RN-164 | Cuando una reconsulta produce un resultado concluyente con estado, monto máximo o autorización distintos, el sistema DEBE crear una nueva versión sin modificar la previa y convertirla en la versión activa. | MUST | INV-18, PA-23 |
| RN-165 | Cuando se confirma una respuesta de preevaluación o autorización adicional, el sistema DEBE auditar actor, instante, correlación y versión. | MUST | INV-09, INV-18 |
| RN-166 | Cuando una nueva versión concluyente de preevaluación reemplaza la activa, el sistema DEBE conservar las simulaciones y aceptaciones anteriores para auditoría, marcarlas inválidas para contratación y transicionar sus simulaciones exitosas a `SIMULACION_INVALIDADA`. | MUST | INV-18, PA-23 |
| RN-167 | Cuando se emite o reemplaza una oferta preferencial, el sistema DEBE asignar un `ofertaId` nuevo e inmutable y conservar la emisión anterior en historial. | MUST | INV-19 |
| RN-168 | Cuando una oferta preferencial está vencida, reemplazada o contratada, el sistema NO DEBE permitir nuevas simulaciones ni contrataciones para su `ofertaId`. | MUST | INV-19 |
| RN-169 | Cuando un cliente tiene varias ofertas preferenciales vigentes, el sistema DEBE exigir que seleccione explícitamente el `ofertaId` que desea simular o contratar. | MUST | INV-19 |
| RN-170 | Cuando se procesa una oferta preferencial, el sistema DEBE tratar cada `ofertaId` como una oportunidad independiente con un máximo de un contrato y sujeta a la línea compartida. | MUST | INV-03, INV-19 |
| RN-171 | Cuando una oferta preferencial vence, se reemplaza o se contrata, el sistema NO DEBE iniciar automáticamente el flujo regular. | MUST | INV-19 |
| RN-172 | Cuando el cliente elige explícitamente iniciar el flujo regular y no existe otra solicitud regular activa, el sistema DEBE crear primero un `solicitudId` en `SOLICITUD_INICIADA` y después obtener una preevaluación vinculada a esa solicitud. | MUST | INV-18, INV-19, PA-38 |
| RN-173 | Mientras un cliente tenga una solicitud regular activa, el sistema NO DEBE crear otra solicitud regular para ese cliente. | MUST | INV-19 |
| RN-174 | Cuando se reconsulta la preevaluación de una solicitud regular activa, el sistema DEBE crear una nueva versión dentro del mismo `solicitudId` sin crear otra oportunidad. | MUST | INV-18, INV-19 |
| RN-175 | Cuando una solicitud regular obtiene un contrato, el sistema DEBE cerrarla y NO DEBE reutilizar su `solicitudId`. | MUST | INV-19 |
| RN-176 | Cuando una solicitud regular cierra sin contrato, el sistema NO DEBE reutilizar su `solicitudId` para un préstamo posterior. | MUST | INV-19 |
| RN-177 | Cuando un cliente inicia otro préstamo regular después del cierre de su solicitud anterior, el sistema DEBE crear un `solicitudId` nuevo y exigir una nueva preevaluación. | MUST | INV-19 |
| RN-178 | Cuando una guarda de negocio incumplida se confirma con evidencia durable, el sistema DEBE resolver el intento como `CONTRATACION_RECHAZADA` y liberar su reserva exactamente una vez. | MUST | PA-36, RN-091 |
| RN-179 | Cuando un fallo técnico definitivo se confirma con evidencia durable, el sistema DEBE resolver el intento como `CONTRATACION_FALLIDA_DEFINITIVA` y liberar su reserva exactamente una vez. | MUST | PA-36, RN-070, RN-091 |
| RN-180 | Cuando un fallo técnico se clasifica como recuperable, el sistema DEBE resolver el intento como `CONTRATACION_FALLIDA_RECUPERABLE`, mantener su reserva y permitir únicamente un reintento controlado del mismo intento. | MUST | PA-36, RN-031 |
| RN-181 | Mientras una contratación permanezca incierta, el sistema DEBE consultar la evidencia durable cada 30 segundos durante un máximo de 15 minutos desde que se registró `CONTRATACION_PENDIENTE`. | MUST | PA-37, INV-13 |
| RN-182 | Cuando una contratación completa 15 minutos en `CONTRATACION_PENDIENTE` sin evidencia concluyente, el sistema DEBE pasarla a `CONTRATACION_EN_REVISION_MANUAL`, mantener su reserva y conservar el bloqueo de otro intento para la oportunidad. | MUST | PA-37, INV-13 |
| RN-183 | Cuando una contratación entra en `CONTRATACION_EN_REVISION_MANUAL`, el sistema DEBE registrar un incidente crítico con objetivo de resolución de 4 horas desde el ingreso a revisión. | MUST | PA-37 |
| RN-184 | Para resolver una contratación pendiente o en revisión manual, el sistema solo DEBE aceptar como evidencia concluyente el conjunto contractual atómico completo, un marcador durable de rollback o cierre sin contrato, o un resultado autoritativo del gestor transaccional que pruebe el rollback. | MUST | PA-37, INV-13 |
| RN-185 | Cuando se incumple el objetivo de 4 horas sin evidencia concluyente, el sistema DEBE mantener `CONTRATACION_EN_REVISION_MANUAL`, conservar la reserva y escalar el incidente; NO DEBE inferir éxito, rechazo ni fallo por el tiempo transcurrido, la ausencia de filas, un timeout o una lectura no disponible. | MUST | PA-37, INV-13 |
| RN-186 | Mientras opere la primera evaluación, el sistema DEBE calcular todos los importes financieros en PEN. | MUST | PA-07 |
| RN-187 | Cuando recibe o persiste una TEA, el sistema DEBE representarla como valor decimal, donde 20 % corresponde a `0.20`. | MUST | PA-07 |
| RN-188 | Cuando construye cada período anterior al último, el sistema DEBE redondear primero el interés a dos decimales, calcular el capital como cuota financiera menos interés y calcular el saldo final como saldo inicial menos capital. | MUST | PA-07 |
| RN-189 | Cuando construye la última cuota, el sistema DEBE usar como capital el saldo pendiente a dos decimales, calcular el interés sobre ese saldo y ajustar la cuota financiera final a la suma de capital e interés sin modificar el seguro fijo. | MUST | PA-07 |
| RN-190 | Cuando calcula la TCEA, el sistema DEBE usar como flujo inicial positivo el desembolso y como flujos negativos los pagos mensuales realmente redondeados, incluido el seguro. | MUST | PA-07 |
| RN-191 | Cuando calcula la TIR mensual, el sistema DEBE aplicar bisección en `[-0.999999999999, 1000000]`, con tolerancia `1E-12` y máximo de 1 000 iteraciones; si no existe cambio de signo o no converge, DEBE aplicar RN-063 y RN-064. | MUST | PA-07 |
| RN-192 | Cuando calcula la TCEA, el sistema DEBE conservarla con 12 decimales y presentarla como porcentaje con dos decimales usando `HALF_UP`. | MUST | PA-07 |
| RN-193 | Cuando genera las fechas del cronograma, el sistema DEBE usar como ancla la fecha de creación de la simulación y el mismo día en cada mes; si el día no existe, DEBE usar el último día del mes, y si el ancla es fin de mes, DEBE conservar fin de mes. | MUST | PA-07 |
| RN-194 | Cuando calcula intereses o TCEA, el sistema DEBE tratar los períodos como meses financieros uniformes y NO DEBE prorratear por diferencias de días calendario. | MUST | PA-07 |
| RN-195 | Mientras opere la primera evaluación, el sistema DEBE aceptar montos solicitados desde S/ 500 hasta S/ 10 000 inclusive y únicamente en incrementos de S/ 100. | MUST | PA-08 |
| RN-196 | Mientras opere la primera evaluación, el sistema DEBE aceptar únicamente plazos de 6, 12, 18 o 24 meses. | MUST | PA-08 |
| RN-197 | Mientras opere la primera evaluación, el sistema DEBE aceptar una TEA entre `0.00` y `1.00` inclusive y usar `0.20` como TEA pizarra inicial. | MUST | PA-08 |
| RN-198 | Mientras opere la primera evaluación, el sistema DEBE usar un seguro fijo inicial de S/ 10 por cuota y aceptar su configuración únicamente entre S/ 0 y S/ 100 inclusive. | MUST | PA-08 |
| RN-199 | Cuando se configura una cuenta sintética, el sistema DEBE exigir límite de línea, moneda PEN, consumo actual, reservas vigentes y versión de línea; el límite DEBE ser mayor que S/ 0 y no superar S/ 50 000. | MUST | PA-08, DEP-01 |
| RN-200 | Cuando falta un dato obligatorio de la línea o configuración financiera, el sistema DEBE bloquear la simulación o contratación afectada y NO DEBE completar el dato con un valor predeterminado. | MUST | PA-08, INV-18 |
| RN-201 | Cuando se crea una simulación, el sistema DEBE congelar en su versión la TEA y el seguro aplicados; un cambio posterior de configuración solo se aplica a simulaciones nuevas. | MUST | PA-08, PRINCIPIO-005 |
| RN-202 | Cuando se crea la reserva, el sistema DEBE revalidar el límite, saldo disponible y versión vigente de la línea; un cambio puede impedir contratar una simulación anterior, pero NO DEBE modificar sus condiciones congeladas. | MUST | PA-08, RN-139 |
| RN-203 | Cuando `OFERTA_BASE_HABILITADA` crea una oferta base, el sistema DEBE usar como vigencia de esa oferta el intervalo de la versión de preevaluación que la habilitó. | MUST | PA-23, RN-155 |
| RN-204 | Cuando la oferta base requiere autorización adicional, el sistema DEBE exigir que la autorización esté `AUTORIZADA` y que no haya vencido la versión de preevaluación asociada. | MUST | PA-23, RN-157–160 |
| RN-205 | Cuando una reconsulta devuelve `NO_DISPONIBLE`, el sistema NO DEBE reemplazar ni invalidar una versión concluyente que todavía esté vigente. | MUST | PA-23, RN-162 |
| RN-206 | Cuando una nueva versión concluyente reemplaza la preevaluación activa, el sistema NO DEBE permitir contratar simulaciones o aceptaciones vinculadas a la versión reemplazada. | MUST | PA-23, RN-164, RN-166 |
| RN-207 | Cuando cambia la tasa pizarra o el seguro configurado, el sistema NO DEBE invalidar simulaciones existentes y DEBE aplicar el nuevo valor únicamente a simulaciones nuevas. | MUST | PA-23, RN-201 |
| RN-208 | Cuando cambia el límite, saldo o versión de línea, el sistema NO DEBE invalidar una simulación existente por ese solo hecho y DEBE aplicar RN-202 al reservar. | MUST | PA-23 |
| RN-209 | Cuando una oferta preferencial es reemplazada, el sistema DEBE conservar para auditoría sus simulaciones y aceptaciones, marcarlas inválidas para contratación y transicionar sus simulaciones exitosas a `SIMULACION_INVALIDADA`. | MUST | PA-23, RN-167–168 |
| RN-210 | Cuando una solicitud regular pasa a `SOLICITUD_VALIDANDO`, el sistema DEBE solicitar la preevaluación usando el `solicitudId` ya creado y la identidad del cliente autenticado. | MUST | PA-38, RN-154, RN-172 |
| RN-211 | Cuando la preevaluación devuelve `OFERTA_BASE_HABILITADA` válida, el sistema DEBE transicionar la solicitud a `SOLICITUD_HABILITADA`. | MUST | PA-38, RN-156 |
| RN-212 | Cuando la preevaluación devuelve `EVALUACION_ADICIONAL_REQUERIDA`, el sistema DEBE transicionar la solicitud a `SOLICITUD_ESPERANDO_AUTORIZACION`. | MUST | PA-38, RN-157–158 |
| RN-213 | Cuando una autorización vigente cambia a `AUTORIZADA`, el sistema DEBE transicionar la solicitud asociada de `SOLICITUD_ESPERANDO_AUTORIZACION` a `SOLICITUD_HABILITADA`. | MUST | PA-38, RN-160 |
| RN-214 | Cuando una autorización cambia a `DENEGADA` o la preevaluación devuelve `SIN_OFERTA`, el sistema DEBE transicionar la solicitud asociada a `SOLICITUD_RECHAZADA`. | MUST | PA-38, RN-159, RN-161 |
| RN-215 | Cuando la preevaluación devuelve `NO_DISPONIBLE`, el sistema DEBE transicionar la solicitud a `SOLICITUD_PENDIENTE` y permitir reconsultar dentro del mismo `solicitudId`. | MUST | PA-38, RN-162, RN-174 |
| RN-216 | Mientras una solicitud regular no esté en `SOLICITUD_HABILITADA`, el sistema NO DEBE permitir crear una simulación para esa solicitud. | MUST | PA-38 |
| RN-217 | Cuando un fallo técnico de solicitud se clasifica como recuperable, el sistema DEBE registrar `SOLICITUD_FALLIDA_RECUPERABLE`; si se confirma como definitivo, DEBE registrar `SOLICITUD_FALLIDA_DEFINITIVA`. | MUST | PA-22, RN-066, RN-070 |
| RN-218 | Cuando un fallo técnico de simulación se clasifica como recuperable, el sistema DEBE registrar `SIMULACION_FALLIDA_RECUPERABLE`; si se confirma como definitivo, DEBE registrar `SIMULACION_FALLIDA_DEFINITIVA`. | MUST | PA-22, RN-063, RN-066, RN-070 |
| RN-219 | Cuando se recupera una solicitud pendiente, el sistema DEBE transicionar según la evidencia a `SOLICITUD_HABILITADA`, `SOLICITUD_ESPERANDO_AUTORIZACION`, `SOLICITUD_RECHAZADA`, `SOLICITUD_FALLIDA_RECUPERABLE` o `SOLICITUD_FALLIDA_DEFINITIVA`. | MUST | PA-22 |
| RN-220 | Cuando se recupera una simulación pendiente, el sistema DEBE transicionar según la evidencia a `SIMULACION_EXITOSA`, `SIMULACION_FALLIDA_RECUPERABLE` o `SIMULACION_FALLIDA_DEFINITIVA`. | MUST | PA-22 |
| RN-221 | Cuando se reintenta una solicitud en `SOLICITUD_FALLIDA_RECUPERABLE`, el sistema DEBE reutilizar el mismo `solicitudId` y transicionar a `SOLICITUD_VALIDANDO`. | MUST | PA-22, RN-031 |
| RN-222 | Cuando se reintenta una simulación en `SIMULACION_FALLIDA_RECUPERABLE`, el sistema DEBE reutilizar el mismo `simulacionId` y transicionar a `SIMULACION_CALCULANDO`. | MUST | PA-22, RN-031 |
| RN-223 | Cuando RN-085 detecta un vencimiento en `CONTRATACION_VALIDANDO` o `CONTRATACION_REGISTRANDO`, el sistema DEBE revertir cualquier escritura no confirmada, transicionar a `CONTRATACION_RECHAZADA` y liberar la reserva exactamente una vez. | MUST | PA-22, RN-091, RN-096 |
| RN-224 | Cuando ocurre un fallo técnico recuperable confirmado en `CONTRATACION_VALIDANDO` o `CONTRATACION_REGISTRANDO`, el sistema DEBE transicionar a `CONTRATACION_FALLIDA_RECUPERABLE`. | MUST | PA-22, RN-180 |
| RN-225 | Cuando ocurre un fallo técnico definitivo confirmado en `CONTRATACION_VALIDANDO` o `CONTRATACION_REGISTRANDO`, el sistema DEBE transicionar a `CONTRATACION_FALLIDA_DEFINITIVA`. | MUST | PA-22, RN-179 |
| RN-226 | Cuando el resultado técnico queda incierto en `CONTRATACION_VALIDANDO` o `CONTRATACION_REGISTRANDO`, el sistema DEBE transicionar a `CONTRATACION_PENDIENTE`. | MUST | PA-22, RN-049 |
| RN-227 | Cuando un error se clasifica como recuperable, el sistema DEBE permitir únicamente reintentos controlados de la misma operación dentro de los límites aprobados y auditar cada intento. | MUST | PA-03, RN-031 |
| RN-228 | Cuando un error se clasifica como definitivo, el sistema NO DEBE reintentar automáticamente la operación. | MUST | PA-03, RN-070 |
| RN-229 | Cuando un resultado se clasifica como incierto, el sistema DEBE mantenerlo pendiente hasta obtener evidencia concluyente y NO DEBE convertirlo en fallo solo por agotar consultas o reintentos. | MUST | PA-03 |
| RN-230 | Cuando un error recuperable agota sus reintentos controlados, el sistema solo DEBE reclasificarlo como definitivo si existe evidencia concluyente; en caso contrario, DEBE conservarlo pendiente y escalarlo conforme a la operación. | MUST | PA-03 |
| RN-231 | Cuando un desembolso no se resuelve mediante la recuperación automática aprobada, el sistema DEBE transicionarlo a `DESEMBOLSO_EN_REVISION_MANUAL`. | MUST | PA-12, RN-025 |
| RN-232 | Mientras un desembolso esté en revisión manual, solo un actor con rol `OPERADOR_REVISION_DESEMBOLSO` DEBE poder consultar su evidencia, registrar observaciones o solicitar otra consulta de la operación original. | MUST | PA-12, PA-15 |
| RN-233 | Mientras un desembolso esté en revisión manual, el operador NO DEBE crear otra operación de desembolso, modificar un terminal confirmado ni declarar un resultado sin evidencia concluyente del core simulado. | MUST | PA-12, INV-05, INV-07 |
| RN-234 | Cuando la revisión manual obtiene y persiste una respuesta `EXITOSA`, `RECHAZADA` o `FALLIDA_DEFINITIVA`, el sistema DEBE resolver el desembolso conforme a RN-145–147; sin esa evidencia, DEBE mantenerlo en revisión. | MUST | PA-12 |
| RN-235 | Cuando un actor realiza una acción de revisión manual, el sistema DEBE auditar su identidad, instante, motivo, evidencia consultada, correlación, versión y resultado. | MUST | PA-12, INV-09 |
| RN-236 | Cuando una evidencia contradice un terminal local confirmado, el sistema DEBE crear un expediente de conflicto separado y NO DEBE modificar el terminal. | MUST | PA-12, RN-152 |
| RN-237 | El rol `CLIENTE_SINTETICO` solo DEBE poder consultar y operar ofertas, solicitudes, simulaciones, aceptaciones, contrataciones y desembolsos propios. | MUST | PA-15 |
| RN-238 | El rol `OPERADOR_REVISION_DESEMBOLSO` solo DEBE poder actuar sobre casos en revisión dentro de las facultades de RN-232–235 y NO DEBE acceder a credenciales ni crear contratos. | MUST | PA-15 |
| RN-239 | El rol `AUDITOR` solo DEBE poder consultar estados y registros de auditoría autorizados y NO DEBE producir transiciones de negocio. | MUST | PA-15 |
| RN-240 | Un proceso con rol `SERVICIO_INTERNO` solo DEBE poder ejecutar las acciones automáticas expresamente asignadas a su función. | MUST | PA-15 |
| RN-241 | Cuando la sesión de un actor está vencida o fue cerrada, el sistema NO DEBE permitir aceptar, contratar ni ejecutar otra acción protegida hasta una nueva autenticación válida. | MUST | PA-15 |
| RN-242 | Cuando se acumulan intentos reiterados de autenticación fallida, el sistema DEBE aplicar un bloqueo temporal conforme a la política de seguridad aprobada. | MUST | PA-15 |
| RN-243 | Cuando un actor intenta consultar u operar un recurso ajeno o no autorizado, el sistema NO DEBE revelar su existencia ni contenido. | MUST | PA-15, RN-122 |
| RN-244 | Antes de integrar un cambio, el sistema DEBE contar con evidencia de verificación de los controles aplicables de OWASP ASVS 5.0.0 nivel 1. | MUST | PA-16, PRINCIPIO-012 |
| RN-245 | Mientras exista un hallazgo de seguridad crítico o alto abierto, el cambio NO DEBE integrarse. | MUST | PA-16 |
| RN-246 | Cuando se acepta un riesgo medio, el sistema DEBE conservar la aprobación humana documentada, su responsable y la evidencia de la evaluación. | MUST | PA-16 |
| RN-247 | Cuando un control de seguridad se declara no aplicable, el sistema DEBE conservar su justificación aprobada. | MUST | PA-16 |
| RN-248 | Cuando se repite una petición de contratación con la misma clave y datos equivalentes, el sistema DEBE devolver el intento original y NO DEBE crear otro intento, contrato ni desembolso. | MUST | PA-29, RN-115 |
| RN-249 | Cuando una clave de contratación se reutiliza con datos distintos, el sistema DEBE responder como conflicto sin ejecutar la petición. | MUST | PA-29, RN-067, RN-116 |
| RN-250 | Cuando se usa una clave nueva, el sistema DEBE aplicar igualmente las guardas de contrato existente, intento incierto y saldo disponible. | MUST | PA-29, RN-117 |
| RN-251 | Cuando dos clientes usan el mismo valor de clave, el sistema DEBE mantener sus ámbitos separados y NO DEBE permitir que uno recupere o conozca la operación del otro. | MUST | PA-29, RN-120 |
| RN-252 | La equivalencia técnica de datos, respuestas y representación de la clave idempotente DEBEN documentarse en OpenAPI y ADR sin alterar las reglas funcionales RN-248–251. | MUST | PA-29, PRINCIPIO-014 |
| RN-253 | Cuando se presenta una simulación al cliente, el sistema DEBE conservar evidencia de que la versión íntegra con todas las condiciones obligatorias estuvo disponible para ese cliente antes de aceptar. | MUST | PA-30, PRINCIPIO-004 |
| RN-254 | Cuando el cliente acepta, el sistema DEBE verificar que la sesión sea válida y que la simulación sea propia, exitosa, vigente, no invalidada y exactamente la misma versión presentada. | MUST | PA-30 |
| RN-255 | Cuando la sesión está vencida o la versión está vencida, invalidada, desactualizada o no coincide con la presentada, el sistema NO DEBE registrar la aceptación y DEBE informar el motivo funcional aplicable sin revelar recursos ajenos. | MUST | PA-30 |
| RN-256 | Cuando se repite la aceptación de la misma versión por el mismo cliente, el sistema DEBE devolver la aceptación existente sin duplicarla. | MUST | PA-30, RN-092 |
| RN-257 | La aceptación DEBE conservar cliente, simulación, versión íntegra presentada, instante, acción expresa y vínculo con la evidencia de presentación. | MUST | PA-30, RN-105 |
| RN-258 | El mecanismo técnico de integridad de la versión y las respuestas de la API DEBEN documentarse en OpenAPI o ADR sin sustituir la evidencia funcional exigida por RN-253–257. | MUST | PA-30, PRINCIPIO-014 |

## 7. Cuatro máquinas de estados

Cada fila conserva el catálogo funcional aprobado o aplica una decisión de las rondas QA. Las guardas incompletas se indican con su PA. En todas las filas aplican RN-069, RN-081 y RN-082. Los nombres de estado se usan completos. Las matrices documentan las transiciones conocidas.

### Solicitud

| Origen | Evento o guarda | Destino | Fuente |
|---|---|---|---|
| — | Inicio de solicitud | SOLICITUD_INICIADA | DEC-58 |
| SOLICITUD_INICIADA | Inicio de validación | SOLICITUD_VALIDANDO | DEC-58 |
| SOLICITUD_VALIDANDO | `OFERTA_BASE_HABILITADA` válida | SOLICITUD_HABILITADA | RN-211 |
| SOLICITUD_VALIDANDO | `EVALUACION_ADICIONAL_REQUERIDA` | SOLICITUD_ESPERANDO_AUTORIZACION | RN-212 |
| SOLICITUD_ESPERANDO_AUTORIZACION | Autorización vigente `AUTORIZADA` | SOLICITUD_HABILITADA | RN-213 |
| SOLICITUD_ESPERANDO_AUTORIZACION | Autorización `DENEGADA` | SOLICITUD_RECHAZADA | RN-214 |
| SOLICITUD_VALIDANDO | `SIN_OFERTA` | SOLICITUD_RECHAZADA | RN-214 |
| SOLICITUD_VALIDANDO | `NO_DISPONIBLE` | SOLICITUD_PENDIENTE | RN-215 |
| SOLICITUD_VALIDANDO | Fallo técnico recuperable confirmado | SOLICITUD_FALLIDA_RECUPERABLE | RN-217 |
| SOLICITUD_VALIDANDO | Fallo técnico definitivo confirmado | SOLICITUD_FALLIDA_DEFINITIVA | RN-217 |
| SOLICITUD_VALIDANDO | Resultado técnico incierto distinto de `NO_DISPONIBLE` | SOLICITUD_PENDIENTE | DEC-58 |
| SOLICITUD_FALLIDA_RECUPERABLE | Reintento controlado de la misma solicitud | SOLICITUD_VALIDANDO | RN-221, PA-03 |
| SOLICITUD_PENDIENTE | Reconsulta de preevaluación | SOLICITUD_VALIDANDO | RN-110, RN-174, RN-215 |
| SOLICITUD_PENDIENTE | Evidencia de oferta base habilitada | SOLICITUD_HABILITADA | RN-219 |
| SOLICITUD_PENDIENTE | Evidencia de evaluación adicional requerida | SOLICITUD_ESPERANDO_AUTORIZACION | RN-219 |
| SOLICITUD_PENDIENTE | Evidencia de rechazo | SOLICITUD_RECHAZADA | RN-219 |
| SOLICITUD_PENDIENTE | Evidencia de fallo técnico recuperable | SOLICITUD_FALLIDA_RECUPERABLE | RN-219 |
| SOLICITUD_PENDIENTE | Evidencia de fallo técnico definitivo | SOLICITUD_FALLIDA_DEFINITIVA | RN-219 |
| SOLICITUD_INICIADA o SOLICITUD_HABILITADA | Cancelación permitida | SOLICITUD_CANCELADA | DEC-58, PA-13 |

### Simulación

| Origen | Evento o guarda | Destino | Fuente |
|---|---|---|---|
| — | Solicitud de simulación | SIMULACION_SOLICITADA | DEC-58 |
| SIMULACION_SOLICITADA | Inicio del cálculo | SIMULACION_CALCULANDO | DEC-58 |
| SIMULACION_CALCULANDO | Cálculo completo, válido y almacenado | SIMULACION_EXITOSA | DEC-58, RN-083 |
| SIMULACION_CALCULANDO | Fallo técnico recuperable confirmado | SIMULACION_FALLIDA_RECUPERABLE | RN-218 |
| SIMULACION_CALCULANDO | Fallo técnico definitivo o TCEA inválida | SIMULACION_FALLIDA_DEFINITIVA | RN-063, RN-218 |
| SIMULACION_CALCULANDO | Resultado incierto | SIMULACION_PENDIENTE | DEC-58 |
| SIMULACION_FALLIDA_RECUPERABLE | Reintento controlado de la misma simulación | SIMULACION_CALCULANDO | RN-222, PA-03 |
| SIMULACION_PENDIENTE | Evidencia de cálculo completo, válido y almacenado | SIMULACION_EXITOSA | RN-220 |
| SIMULACION_PENDIENTE | Evidencia de fallo técnico recuperable | SIMULACION_FALLIDA_RECUPERABLE | RN-220 |
| SIMULACION_PENDIENTE | Evidencia de fallo técnico definitivo | SIMULACION_FALLIDA_DEFINITIVA | RN-220 |
| SIMULACION_EXITOSA | Se alcanza el límite de 30 minutos UTC | SIMULACION_EXPIRADA | RN-057, RN-074 |
| SIMULACION_EXITOSA | Nueva preevaluación concluyente reemplaza la versión asociada | SIMULACION_INVALIDADA | RN-164, RN-166, RN-206 |
| SIMULACION_EXITOSA | Reemplazo de la oferta preferencial asociada | SIMULACION_INVALIDADA | RN-167–168, RN-209 |

Pueden coexistir simulaciones. Generar otra no invalida por sí solo una aceptación previa. Cada intento contractual debe identificar su simulación y su aceptación y comprobar vigencia y crédito disponible. PA-23 conserva las demás causas de invalidación pendientes.

### Contratación

| Origen | Evento o guarda | Destino | Fuente |
|---|---|---|---|
| — | Confirmación atómica de un nuevo intento permitido y su reserva | CONTRATACION_INICIADA | RN-034, RN-048, RN-087, RN-107–108 |
| CONTRATACION_INICIADA | Inicio de validación | CONTRATACION_VALIDANDO | DEC-58 |
| CONTRATACION_VALIDANDO | Validaciones satisfechas | CONTRATACION_REGISTRANDO | DEC-58 |
| CONTRATACION_VALIDANDO | Incumplimiento confirmado de una guarda de negocio o vencimiento | CONTRATACION_RECHAZADA | RN-178, RN-223 |
| CONTRATACION_REGISTRANDO | Contrato, auditoría, orden y conversión de reserva confirmados atómicamente | CONTRATACION_EXITOSA | RN-016, RN-090, RN-099 |
| CONTRATACION_REGISTRANDO | Vencimiento en comprobación final y reversión de escrituras no confirmadas | CONTRATACION_RECHAZADA | RN-223 |
| CONTRATACION_VALIDANDO o CONTRATACION_REGISTRANDO | Fallo técnico recuperable confirmado | CONTRATACION_FALLIDA_RECUPERABLE | RN-224 |
| CONTRATACION_VALIDANDO o CONTRATACION_REGISTRANDO | Fallo técnico definitivo confirmado | CONTRATACION_FALLIDA_DEFINITIVA | RN-225 |
| CONTRATACION_VALIDANDO o CONTRATACION_REGISTRANDO | Resultado técnico incierto | CONTRATACION_PENDIENTE | RN-226 |
| CONTRATACION_PENDIENTE | Lectura durable confirma el conjunto contractual atómico | CONTRATACION_EXITOSA | RN-045, RN-050, RN-125–126 |
| CONTRATACION_PENDIENTE | Evidencia durable confirma incumplimiento de negocio | CONTRATACION_RECHAZADA | RN-127, RN-178 |
| CONTRATACION_PENDIENTE | Evidencia durable confirma fallo técnico recuperable | CONTRATACION_FALLIDA_RECUPERABLE | RN-127, RN-180 |
| CONTRATACION_PENDIENTE | Evidencia durable confirma fallo técnico definitivo | CONTRATACION_FALLIDA_DEFINITIVA | RN-127, RN-179 |
| CONTRATACION_PENDIENTE | 15 minutos sin evidencia concluyente | CONTRATACION_EN_REVISION_MANUAL | RN-181–182 |
| CONTRATACION_EN_REVISION_MANUAL | Evidencia durable confirma el conjunto contractual atómico | CONTRATACION_EXITOSA | RN-126, RN-184 |
| CONTRATACION_EN_REVISION_MANUAL | Evidencia durable confirma incumplimiento de negocio | CONTRATACION_RECHAZADA | RN-178, RN-184 |
| CONTRATACION_EN_REVISION_MANUAL | Evidencia durable confirma fallo técnico recuperable | CONTRATACION_FALLIDA_RECUPERABLE | RN-180, RN-184 |
| CONTRATACION_EN_REVISION_MANUAL | Evidencia durable confirma fallo técnico definitivo | CONTRATACION_FALLIDA_DEFINITIVA | RN-179, RN-184 |
| CONTRATACION_EN_REVISION_MANUAL | 4 horas sin evidencia concluyente | Permanece en CONTRATACION_EN_REVISION_MANUAL con incidente escalado | RN-183–185 |
| CONTRATACION_FALLIDA_RECUPERABLE | Reintento controlado del mismo intento | CONTRATACION_VALIDANDO | RN-031, RN-180 |

La matriz distingue el rechazo de negocio, el fallo técnico recuperable, el fallo técnico definitivo y la incertidumbre. El vencimiento puede rechazarse durante validación o en la comprobación final de registro. Crear otro intento exige RN-048.

### Desembolso

| Origen | Evento o guarda | Destino | Fuente |
|---|---|---|---|
| — | Confirmación del contrato y de la orden única | DESEMBOLSO_SOLICITADO | RN-042, RN-099 |
| DESEMBOLSO_SOLICITADO | Adquisición exclusiva y confirmación durable previa a la primera entrega | DESEMBOLSO_PROCESANDO | RN-100, RN-130–132 |
| DESEMBOLSO_PROCESANDO, DESEMBOLSO_CONSULTANDO_ESTADO o DESEMBOLSO_EN_REVISION_MANUAL | `EXITOSA` y persistencia local confirmada | DESEMBOLSO_SIMULADO_EXITOSO | RN-020, RN-145 |
| DESEMBOLSO_PROCESANDO, DESEMBOLSO_CONSULTANDO_ESTADO o DESEMBOLSO_EN_REVISION_MANUAL | `FALLIDA_DEFINITIVA`, evidencia exigida y persistencia local confirmada | DESEMBOLSO_FALLIDO | RN-053, RN-147 |
| DESEMBOLSO_PROCESANDO, DESEMBOLSO_CONSULTANDO_ESTADO o DESEMBOLSO_EN_REVISION_MANUAL | `RECHAZADA` y persistencia local confirmada | DESEMBOLSO_RECHAZADO | RN-146, RN-150 |
| DESEMBOLSO_PROCESANDO | Resultado ambiguo, timeout o persistencia local sin confirmar | DESEMBOLSO_PENDIENTE_DE_GESTION | RN-021, RN-022, RN-051, RN-149 |
| DESEMBOLSO_PENDIENTE_DE_GESTION | Consulta de la operación original | DESEMBOLSO_CONSULTANDO_ESTADO | RN-023 |
| DESEMBOLSO_CONSULTANDO_ESTADO | Consulta inconclusa, incluida operación no encontrada | DESEMBOLSO_PENDIENTE_DE_GESTION | RN-052 |
| DESEMBOLSO_PENDIENTE_DE_GESTION | Reentrega idempotente del comando original con idéntico identificador y datos | DESEMBOLSO_PROCESANDO | RN-024, RN-133, RN-148 |
| DESEMBOLSO_PENDIENTE_DE_GESTION | Recuperación automática aprobada finaliza sin resolución | DESEMBOLSO_EN_REVISION_MANUAL | RN-025, RN-231 |
| DESEMBOLSO_EN_REVISION_MANUAL | Evidencia insuficiente | Permanece en DESEMBOLSO_EN_REVISION_MANUAL con resultado pendiente | RN-072 |

Las respuestas del core y su correspondencia local se fijan en RN-144–153. La revisión manual se rige por RN-231–236. El estado pendiente es lógico cuando no es posible persistirlo inmediatamente. Desde `DESEMBOLSO_PROCESANDO`, la recuperación consulta la operación original y puede reentregar técnicamente el mismo comando conforme a RN-133 y RN-148; nunca crea otro identificador ni otra operación lógica. Los límites operativos de consulta y reentrega se documentan en la política técnica sin alterar estas reglas funcionales.

## 8. Terminalidad

- Son terminales confirmados `SOLICITUD_RECHAZADA`, `SOLICITUD_CANCELADA`, `SOLICITUD_FALLIDA_DEFINITIVA`, `SIMULACION_EXPIRADA`, `SIMULACION_INVALIDADA`, `SIMULACION_FALLIDA_DEFINITIVA`, `CONTRATACION_EXITOSA`, `CONTRATACION_RECHAZADA`, `CONTRATACION_FALLIDA_DEFINITIVA`, `DESEMBOLSO_SIMULADO_EXITOSO` y `DESEMBOLSO_RECHAZADO`. `DESEMBOLSO_FALLIDO` definitivo exige RN-053.
- Los estados en curso, pendientes y de revisión son no terminales. `SIMULACION_EXITOSA` puede expirar o invalidarse. `CONTRATACION_FALLIDA_RECUPERABLE` y `CONTRATACION_EN_REVISION_MANUAL` son no terminales y conservan la reserva. Todo fallo definitivo exige evidencia y es terminal.
- Se rechaza toda transición desde un terminal confirmado. Los ajustes llevan identidad independiente. Aplican INV-07 y RN-033.

## 9. Derivaciones técnicas

- La serialización usa el orden lógico obligatorio oportunidad–línea y exige revalidar guardas, estado y versión después de toda espera. La prioridad de contratación actúa desde la adquisición del bloqueo de oportunidad y preserva las aceptaciones ya confirmadas. La aceptación de otra simulación no sustituye una aceptación anterior. Las repeticiones con el mismo cliente, simulación y versión inmutable se deduplican. El mecanismo físico equivalente se documenta en ADR y debe satisfacer RN-136–143.
- Una simulación posterior no prolonga la vigencia de una oferta. La aceptación identifica la versión íntegra y solo puede autorizar la contratación mientras sus condiciones sean válidas. PA-23 conserva los detalles pendientes.
- En RN-019, la duplicidad rechaza un segundo comando de ejecución. RN-043 permite recuperar el resultado con la misma clave y datos. Una clave nueva tampoco habilita un segundo desembolso.

Trazabilidad de decisiones y QA en [05-preguntas-abiertas.md](05-preguntas-abiertas.md).
