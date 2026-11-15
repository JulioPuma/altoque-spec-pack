# Altoque — Historias de Usuario

**Archivo:** `06-user-stories.md`  
**Versión del Spec Pack:** 0.2  
**País de operación:** Perú  
**Ámbito:** Primera evaluación en entorno cerrado con cuentas y datos sintéticos

## Épica 1 — Acceso seguro

### HU-001: Autenticarse con una cuenta sintética

**Prioridad:** Must  
**Como** cliente sintético, **quiero** autenticarme con mis credenciales precargadas, **para** acceder de forma segura a mis operaciones crediticias.

**Criterios de aceptación:**

- DADO QUE existe una cuenta sintética activa cuyo DNI sintético y contraseña independiente coinciden con las credenciales ingresadas, CUANDO el cliente inicia sesión, ENTONCES el sistema crea una sesión autenticada y permite consultar únicamente los recursos de ese cliente.
- DADO QUE la contraseña ingresada no coincide con la cuenta sintética, CUANDO el cliente intenta iniciar sesión, ENTONCES el sistema rechaza el acceso y no revela ofertas, solicitudes, simulaciones, contrataciones ni desembolsos.
- DADO QUE la sesión venció o fue cerrada, CUANDO el cliente intenta aceptar, contratar o consultar una operación protegida, ENTONCES el sistema exige una nueva autenticación válida antes de ejecutar la acción.
- [NUEVO] DADO QUE se intenta autenticar una cuenta no precargada, inactiva o perteneciente a una persona real, CUANDO se envían las credenciales, ENTONCES el sistema rechaza el acceso sin revelar cuál dato falló ni crear una sesión.
- [NUEVO] DADO QUE se alcanza exactamente el umbral de intentos fallidos definido por la política de seguridad aprobada, CUANDO se realiza el intento que completa el umbral, ENTONCES el sistema aplica el bloqueo temporal y audita el evento sin exponer credenciales.

**Reglas de negocio relacionadas:** RN-001, RN-002, RN-121, RN-122, RN-124, RN-237, RN-241, RN-242, RN-243  
**Notas:** Trazabilidad de alcance: M-01. No incluye registro autónomo, personas reales ni integración con RENIEC. La contraseña sintética no puede ser igual ni derivarse del DNI sintético. Los controles principales deben operar por teclado y tener etiquetas identificables conforme a RN-028 y RN-080.

## Épica 2 — Oportunidades de crédito

### HU-002: Elegir una oferta preferencial vigente

**Prioridad:** Must  
**Como** cliente sintético con una o más ofertas preferenciales, **quiero** elegir una oferta propia vigente, **para** simular el crédito correspondiente a esa oportunidad.

**Criterios de aceptación:**

- DADO QUE el cliente tiene dos ofertas preferenciales propias vigentes con `ofertaId` distintos, CUANDO selecciona una de ellas, ENTONCES el sistema usa exclusivamente el `ofertaId` elegido como oportunidad de la simulación.
- DADO QUE una oferta alcanzó el instante `emitidaEn + 30 días calendario` medido en UTC, CUANDO el cliente intenta simularla o contratarla, ENTONCES el sistema la considera vencida y bloquea la acción.
- DADO QUE la oferta pertenece a otro cliente, CUANDO se intenta consultar u operar su identificador, ENTONCES el sistema no revela su existencia ni su contenido.
- [NUEVO] DADO QUE una oferta está vigente hasta un límite exclusivo, CUANDO se consulta un instante inmediatamente anterior a `emitidaEn + 30 días calendario` UTC, ENTONCES la oferta todavía puede seleccionarse; en el instante exacto del límite ya no puede seleccionarse, simularse ni contratarse.
- [NUEVO] DADO QUE el cliente no tiene ofertas preferenciales propias vigentes, CUANDO abre la lista de oportunidades preferenciales, ENTONCES el sistema muestra una lista vacía sin exponer ofertas ajenas y permite iniciar el flujo regular si cumple sus guardas.

**Reglas de negocio relacionadas:** RN-002, RN-003, RN-004, RN-074, RN-122, RN-167, RN-168, RN-169, RN-170, RN-243  
**Notas:** Trazabilidad de alcance: recorrido preferencial, M-03. Cada emisión es una oportunidad inmutable y admite como máximo un contrato; las oportunidades del mismo cliente comparten la línea de crédito.

### HU-003: Iniciar y resolver una solicitud regular

**Prioridad:** Must  
**Como** cliente sintético, tenga o no una oferta preferencial vigente, **quiero** iniciar explícitamente una solicitud regular y conocer su preevaluación, **para** saber si puedo continuar a la simulación.

**Criterios de aceptación:**

- DADO QUE el cliente no tiene una solicitud regular activa, CUANDO elige iniciar el flujo regular, ENTONCES el sistema crea primero un `solicitudId` inmutable en `SOLICITUD_INICIADA`, lo transiciona a `SOLICITUD_VALIDANDO` y vincula la preevaluación a ese identificador.
- DADO QUE la preevaluación devuelve `OFERTA_BASE_HABILITADA` con monto máximo positivo y moneda PEN, CUANDO se persiste el resultado, ENTONCES la solicitud queda `SOLICITUD_HABILITADA` hasta `creadoEn + 24 horas` con límite exclusivo UTC y permite simular sin superar dicho máximo.
- DADO QUE la preevaluación devuelve `SIN_OFERTA`, CUANDO se persiste el resultado, ENTONCES la solicitud queda `SOLICITUD_RECHAZADA` y se bloquean la simulación y la contratación de esa solicitud.
- DADO QUE la preevaluación devuelve `NO_DISPONIBLE` o carece de un dato obligatorio, CUANDO se procesa la respuesta, ENTONCES se informa indisponibilidad, no se completa ningún dato por defecto y se permite reconsultar dentro del mismo `solicitudId`.
- [NUEVO] DADO QUE la preevaluación devuelve `EVALUACION_ADICIONAL_REQUERIDA` con `evaluacionId`, CUANDO se persiste el resultado, ENTONCES la solicitud queda `SOLICITUD_ESPERANDO_AUTORIZACION` y no permite simular ni contratar hasta recibir una autorización vigente.
- [NUEVO] DADO QUE la versión de preevaluación cumple exactamente `creadoEn + 24 horas` UTC, CUANDO el cliente intenta simular, ENTONCES el sistema la considera vencida por ser el límite exclusivo y bloquea la acción.
- [NUEVO] DADO QUE ya existe una solicitud regular activa del cliente, CUANDO intenta iniciar otra, ENTONCES el sistema no crea un segundo `solicitudId` y devuelve la solicitud activa sin duplicar la oportunidad.
- [NUEVO] DADO QUE el cliente posee una oferta preferencial vigente, CUANDO elige explícitamente iniciar el flujo regular, ENTONCES el sistema crea o devuelve su única solicitud regular activa sin invalidar la oferta preferencial; ambas oportunidades permanecen independientes y comparten la misma línea de crédito.
- [NUEVO] DADO QUE la solicitud propia está `SOLICITUD_INICIADA` o `SOLICITUD_HABILITADA` y no tiene simulaciones, intentos, reservas ni contrato, CUANDO el cliente la cancela, ENTONCES queda `SOLICITUD_CANCELADA`, la repetición devuelve el mismo resultado, se audita la acción y una solicitud posterior usa otro `solicitudId` y nueva preevaluación.
- [NUEVO] DADO QUE una cancelación compite con la creación de una simulación, CUANDO adquiere el bloqueo de oportunidad y revalida las guardas, ENTONCES solo se confirma la cancelación si todavía no existe ninguna simulación; en caso contrario se rechaza sin alterar la solicitud ni la simulación.

**Reglas de negocio relacionadas:** RN-154, RN-155, RN-156, RN-157, RN-158, RN-161, RN-162, RN-163, RN-171, RN-172, RN-173, RN-211, RN-212, RN-214, RN-215, RN-216, RN-260  
**Notas:** Trazabilidad de alcance: M-02. Solo puede existir una solicitud regular activa por cliente. El flujo regular nunca se inicia automáticamente y su inicio no invalida una oferta preferencial vigente. Una reconsulta crea una versión de preevaluación, no otra oportunidad. El scoring real está fuera de alcance.

## Épica 3 — Simulación y transparencia financiera

### HU-004: Simular condiciones financieras válidas

**Prioridad:** Must  
**Como** cliente sintético con una oportunidad habilitada, **quiero** simular un monto y plazo permitidos, **para** obtener condiciones financieras calculadas de manera consistente.

**Criterios de aceptación:**

- DADO QUE la oportunidad permite el monto solicitado y la línea tiene saldo suficiente, CUANDO el cliente simula un monto entre S/ 500 y S/ 10 000 en incrementos de S/ 100 y un plazo de 6, 12, 18 o 24 meses, ENTONCES el backend calcula y almacena una versión de simulación completa en PEN con estado `SIMULACION_EXITOSA`.
- DADO QUE el cliente solicita S/ 550 o un plazo de 9 meses, CUANDO intenta simular, ENTONCES el sistema rechaza los parámetros y no publica una simulación `SIMULACION_EXITOSA`.
- DADO QUE se crea una simulación con TEA `0.20` y seguro de S/ 10 por cuota, CUANDO cambia posteriormente la configuración, ENTONCES la simulación conserva la TEA y el seguro congelados y el cambio solo se aplica a simulaciones nuevas.
- DADO QUE el cálculo de TIR no encuentra cambio de signo en `[-0.999999999999, 1000000]` o no converge en 1 000 iteraciones con tolerancia `1E-12`, CUANDO finaliza el cálculo, ENTONCES la simulación queda `SIMULACION_FALLIDA_DEFINITIVA` y no publica una TCEA.
- [NUEVO] DADO QUE el cliente solicita exactamente S/ 500 o S/ 10 000, con saldo y máximo de oportunidad suficientes, CUANDO simula con un plazo permitido, ENTONCES el sistema acepta ambos valores límite y calcula las condiciones en el backend.
- [NUEVO] DADO QUE falta la TEA, el seguro, el saldo o la versión de línea obligatoria, CUANDO se solicita la simulación, ENTONCES el sistema la bloquea, no completa valores por defecto y conserva la causa del error.
- [NUEVO] DADO QUE ocurre una interrupción técnica recuperable durante el cálculo, CUANDO se reintenta de forma controlada, ENTONCES el sistema reutiliza el mismo `simulacionId`, audita el reintento y no crea una simulación paralela.

**Reglas de negocio relacionadas:** RN-005, RN-006, RN-060, RN-061, RN-063, RN-066, RN-083, RN-186, RN-191, RN-195, RN-196, RN-197, RN-198, RN-200, RN-201, RN-218, RN-222, RN-227  
**Notas:** Trazabilidad de alcance: M-03 y modelo financiero aprobado. El cálculo usa sistema francés, `BigDecimal`, precisión interna de 34 dígitos y redondeo `HALF_UP`; el frontend no recalcula condiciones.

### HU-005: Revisar y aceptar la versión íntegra de una simulación

**Prioridad:** Must  
**Como** cliente sintético, **quiero** revisar el costo completo y aceptar expresamente una simulación vigente, **para** dejar constancia de las condiciones exactas que deseo contratar.

**Criterios de aceptación:**

- DADO QUE una simulación propia está `SIMULACION_EXITOSA`, CUANDO el cliente abre su detalle, ENTONCES el sistema muestra en una misma versión monto, plazo, cuota financiera, seguro, pago total por cuota, intereses, TCEA con dos decimales, costo total y cronograma mensual.
- DADO QUE la versión íntegra presentada es propia, exitosa, no invalidada y tiene menos de 30 minutos desde su creación, CUANDO el cliente ejecuta la acción expresa de aceptar, ENTONCES el sistema registra cliente, instante, simulación, versión aceptada y evidencia de presentación.
- DADO QUE la simulación alcanzó `creadaEn + 30 minutos` UTC o fue invalidada, CUANDO el cliente intenta aceptarla, ENTONCES el sistema no registra la aceptación e informa el motivo funcional aplicable.
- DADO QUE el mismo cliente repite la aceptación de la misma simulación y versión, CUANDO se procesa la repetición, ENTONCES el sistema devuelve la aceptación existente y conserva un solo registro.
- [NUEVO] DADO QUE falta cualquiera de monto, plazo, cuota financiera, seguro, pago total por cuota, intereses, TCEA, costo total o cronograma, CUANDO se abre el detalle, ENTONCES el sistema no habilita la aceptación y muestra un error explícito de información incompleta.
- [NUEVO] DADO QUE una simulación tiene exactamente 30 minutos desde su creación, CUANDO se intenta aceptar, ENTONCES el sistema la considera vencida por el límite exclusivo y no registra la aceptación.
- [NUEVO] DADO QUE una contratación mantiene el bloqueo de la oportunidad, CUANDO llega una aceptación concurrente, ENTONCES esta espera y, al continuar, revalida versión, vigencias y ausencia de contrato antes de confirmarse.

**Reglas de negocio relacionadas:** RN-007, RN-008, RN-009, RN-010, RN-011, RN-012, RN-029, RN-030, RN-057, RN-075, RN-076, RN-077, RN-092, RN-105, RN-140, RN-141, RN-253, RN-254, RN-255, RN-256, RN-257  
**Notas:** Trazabilidad de alcance: M-04. Aceptar no contrata. Una simulación nueva no invalida por sí sola las simulaciones ni aceptaciones anteriores.

## Épica 4 — Contratación

### HU-006: Crear un intento de contratación con reserva

**Prioridad:** Must  
**Como** cliente sintético con una aceptación vigente, **quiero** seleccionar «Contratar», **para** reservar el crédito disponible e iniciar un único intento contractual trazable.

**Criterios de aceptación:**

- DADO QUE la oferta o solicitud, la simulación y su aceptación están vigentes y no existe contrato ni intento incierto para la oportunidad, CUANDO el cliente selecciona «Contratar», ENTONCES el sistema confirma atómicamente un `intentoId` propio y la reserva del monto antes de iniciar el registro contractual.
- DADO QUE la línea tiene S/ 10 000 disponibles y existen dos intentos concurrentes de S/ 6 000 para oportunidades distintas del mismo cliente, CUANDO ambos intentan reservar, ENTONCES solo uno confirma la reserva y el otro se rechaza por saldo insuficiente tras revalidar la línea.
- DADO QUE el cliente repite la petición con la misma clave idempotente y datos equivalentes, CUANDO el sistema la recibe, ENTONCES devuelve el intento original con su estado actual sin crear otra reserva, contratación ni orden de desembolso.
- DADO QUE la misma clave idempotente se reutiliza con datos distintos, CUANDO el sistema recibe la petición, ENTONCES responde conflicto y no ejecuta la contratación.
- [NUEVO] DADO QUE el monto solicitado es exactamente igual al saldo disponible de la línea compartida, CUANDO se adquieren los bloqueos en orden oportunidad–línea y se revalidan las guardas, ENTONCES la reserva puede confirmarse y el saldo disponible queda en S/ 0.
- [NUEVO] DADO QUE la aceptación, simulación u oportunidad vencen mientras la petición espera un bloqueo, CUANDO obtiene el bloqueo y revalida, ENTONCES el sistema rechaza el intento, no inicia el registro contractual y no deja una reserva activa.
- [NUEVO] DADO QUE dos clientes usan el mismo valor textual de clave idempotente, CUANDO ambos contratan sus propias oportunidades, ENTONCES el sistema mantiene ámbitos separados y ninguno recupera ni conoce la operación del otro.

**Reglas de negocio relacionadas:** RN-034, RN-048, RN-067, RN-071, RN-078, RN-086, RN-087, RN-088, RN-107, RN-108, RN-115, RN-116, RN-117, RN-136, RN-137, RN-138, RN-139, RN-142, RN-143, RN-202, RN-248, RN-249, RN-250, RN-251  
**Notas:** Trazabilidad de alcance: M-05. Cuando se requieren ambos bloqueos, el orden es oportunidad y luego línea; después de esperar se revalidan estado, versiones, vigencias, unicidad y saldo.

### HU-007: Confirmar el contrato de forma atómica y auditable

**Prioridad:** Must  
**Como** cliente sintético con un intento y reserva vigentes, **quiero** que el contrato se registre con las condiciones aceptadas, **para** obtener un resultado contractual íntegro y sin duplicados.

**Criterios de aceptación:**

- DADO QUE el intento conserva una aceptación íntegra y vigente, CUANDO la validación final confirma con reloj UTC que siguen vigentes la oportunidad, la simulación y la aceptación, ENTONCES el sistema usa exactamente esa versión aceptada para registrar el contrato.
- DADO QUE todas las validaciones son satisfactorias, CUANDO se confirma el registro, ENTONCES contrato, auditoría obligatoria, orden única de desembolso y conversión de la reserva en consumo se confirman en una sola transacción y la contratación queda `CONTRATACION_EXITOSA`.
- DADO QUE falla cualquiera de los cuatro componentes de la transacción, CUANDO se revierte el registro, ENTONCES ninguno queda confirmado y el sistema no comunica contratación exitosa.
- DADO QUE ya existe un contrato para el mismo `ofertaId` o `solicitudId`, CUANDO se intenta registrar otro, ENTONCES el sistema rechaza el segundo contrato y no crea otra orden de desembolso.
- [NUEVO] DADO QUE falla la auditoría obligatoria durante el registro, CUANDO se revierte la transacción, ENTONCES el sistema registra por separado el fallo del mismo intento sin confirmar contrato, orden ni consumo de línea.
- [NUEVO] DADO QUE el instante autoritativo de la validación final coincide exactamente con el primer vencimiento aplicable, CUANDO se intenta registrar el contrato, ENTONCES el sistema lo rechaza, revierte escrituras no confirmadas y libera la reserva exactamente una vez.
- [NUEVO] DADO QUE la respuesta al registro se pierde después de confirmar la transacción, CUANDO el cliente consulta el mismo intento, ENTONCES el sistema obtiene evidencia durable del conjunto contractual completo y devuelve el contrato existente sin registrar otro.

**Reglas de negocio relacionadas:** RN-015, RN-016, RN-017, RN-018, RN-035, RN-045, RN-046, RN-047, RN-056, RN-079, RN-085, RN-090, RN-091, RN-096, RN-099, RN-123, RN-125, RN-126, RN-223  
**Notas:** Trazabilidad de alcance: M-05 y M-08. La contratación mantiene identidad y estado independientes de solicitud, simulación y desembolso.

## Épica 5 — Desembolso simulado y recuperación

### HU-008: Ejecutar un único desembolso simulado

**Prioridad:** Must  
**Como** cliente sintético con un contrato confirmado, **quiero** que el desembolso simulado se procese una sola vez, **para** evitar duplicidad de la operación asociada a mi contrato.

**Criterios de aceptación:**

- DADO QUE existe una orden durable en `DESEMBOLSO_SOLICITADO`, CUANDO un despachador la adquiere exclusivamente, ENTONCES confirma `DESEMBOLSO_PROCESANDO` antes de la primera entrega y usa el `operacionId` persistente asociado al contrato.
- DADO QUE dos despachadores compiten por la misma orden, CUANDO intentan adquirirla, ENTONCES solo uno obtiene la adquisición durable y solo una ejecución lógica puede iniciarse.
- DADO QUE el core simulado devuelve `EXITOSA` y la persistencia local se confirma, CUANDO se procesa la respuesta, ENTONCES el desembolso queda `DESEMBOLSO_SIMULADO_EXITOSO`.
- DADO QUE llega un segundo comando de ejecución para el mismo contrato, CUANDO el sistema lo recibe, ENTONCES lo rechaza por duplicidad y no crea otra operación lógica.
- [NUEVO] DADO QUE el core recibe el mismo `operacionId` con exactamente los mismos datos, CUANDO procesa la repetición, ENTONCES devuelve el resultado original sin ejecutar otro desembolso; si los datos difieren, rechaza por conflicto.
- [NUEVO] DADO QUE el core confirma éxito pero falla la persistencia local, CUANDO el sistema maneja la interrupción, ENTONCES conserva el desembolso pendiente y consulta la operación original sin declarar un segundo éxito ni ejecutar otra operación.
- [NUEVO] DADO QUE una respuesta tardía llega después de un resultado terminal confirmado, CUANDO contradice el terminal local, ENTONCES el sistema conserva el terminal, registra ambas evidencias y escala un expediente de conflicto.

**Reglas de negocio relacionadas:** RN-019, RN-020, RN-032, RN-042, RN-051, RN-099, RN-100, RN-129, RN-130, RN-131, RN-132, RN-135, RN-145, RN-151, RN-152, RN-153  
**Notas:** Trazabilidad de alcance: M-06 y M-08. El desembolso es simulado; no existen movimientos de dinero reales en la primera evaluación.

### HU-009: Recuperar un resultado de desembolso inconcluso

**Prioridad:** Must  
**Como** cliente sintético con un desembolso pendiente, **quiero** que el sistema recupere el resultado de la operación original, **para** conocer el desenlace sin provocar un segundo desembolso.

**Criterios de aceptación:**

- DADO QUE la entrega termina en timeout, desconexión, respuesta inválida o estado desconocido, CUANDO el sistema procesa el resultado, ENTONCES no declara éxito ni fallo y conserva `DESEMBOLSO_PENDIENTE_DE_GESTION`.
- DADO QUE el desembolso está pendiente, CUANDO se inicia la recuperación, ENTONCES el sistema consulta la operación original usando el mismo `operacionId` y no crea otro identificador.
- DADO QUE la consulta responde `NO_ENCONTRADA`, CUANDO la política técnica permite reentregar, ENTONCES el sistema reentrega únicamente el comando original con el mismo `operacionId` y exactamente los mismos datos.
- DADO QUE la recuperación automática finaliza sin evidencia concluyente, CUANDO se agota el ciclo aprobado, ENTONCES el desembolso transiciona a `DESEMBOLSO_EN_REVISION_MANUAL` y conserva el resultado pendiente.
- [NUEVO] DADO QUE dos procesos recuperan concurrentemente la misma versión del desembolso, CUANDO intentan confirmar una transición, ENTONCES como máximo uno la confirma y el otro revalida el estado y la versión actuales.
- [NUEVO] DADO QUE la evidencia concluyente acredita que la operación no se ejecutó ni puede completarse, CUANDO se persiste el resultado, ENTONCES el sistema confirma `DESEMBOLSO_FALLIDO`, conserva el contrato y el consumo de línea y no libera consumo sin un ajuste auditable autorizado.
- [NUEVO] DADO QUE la recuperación sigue sin evidencia al ingresar o permanecer en revisión manual, CUANDO transcurre cualquier plazo operativo, ENTONCES el sistema no infiere éxito ni fallo por tiempo y mantiene el resultado pendiente.

**Reglas de negocio relacionadas:** RN-021, RN-022, RN-023, RN-024, RN-025, RN-052, RN-053, RN-054, RN-055, RN-072, RN-102, RN-103, RN-104, RN-133, RN-134, RN-148, RN-149, RN-231  
**Notas:** Trazabilidad de alcance: M-07 y M-08. Un timeout no equivale a fallo. La revisión manual no puede crear otra operación ni modificar un estado terminal sin evidencia concluyente.

## Épica 6 — Seguimiento e instrumentación

### HU-010: Consultar operaciones propias después de reingresar

**Prioridad:** Must  
**Como** cliente sintético, **quiero** consultar mis contrataciones y desembolsos al volver a ingresar, **para** conocer el estado actual de operaciones pendientes o resueltas.

**Criterios de aceptación:**

- DADO QUE el cliente autenticado tiene una contratación `CONTRATACION_PENDIENTE`, CUANDO vuelve a ingresar y consulta sus operaciones, ENTONCES el sistema muestra el mismo `intentoId` y su estado durable actual sin iniciar otro intento.
- DADO QUE el cliente autenticado tiene un desembolso pendiente, en revisión o terminal, CUANDO consulta su detalle, ENTONCES el sistema muestra el `operacionId` persistente y el último estado confirmado.
- DADO QUE el identificador consultado pertenece a otro cliente, CUANDO se solicita su detalle, ENTONCES el sistema no revela su existencia, estado ni contenido.
- DADO QUE una oferta propia se visualiza por primera vez, CUANDO se registra el evento de visualización, ENTONCES se conserva la cohorte de origen; las visualizaciones repetidas de la misma oferta por el mismo cliente no crean otra unidad para el KPI.
- [NUEVO] DADO QUE el cliente autenticado no tiene contrataciones ni desembolsos, CUANDO consulta sus operaciones, ENTONCES el sistema devuelve una lista vacía sin error y sin mostrar recursos de otros clientes.
- [NUEVO] DADO QUE una operación cambia de estado después de que el cliente abrió la consulta, CUANDO actualiza el detalle, ENTONCES el sistema muestra el último estado y versión confirmados sin producir una transición de negocio.
- [NUEVO] DADO QUE ocurre una falla de red al consultar, CUANDO no puede obtenerse una lectura durable, ENTONCES el sistema informa indisponibilidad sin presentar un estado supuesto ni modificar la operación.
- [NUEVO] DADO QUE el backend confirma una primera visualización de oportunidad, un cambio de parámetros, una consulta de cronograma, una aceptación o un contrato, CUANDO conserva la evidencia KPI, ENTONCES registra el evento canónico de RN-261 con identidad, instante UTC, correlación, fuente, versión, esquema y deduplicación, sin calcular fórmulas todavía abiertas.
- [NUEVO] DADO QUE se repite técnicamente el mismo evento o llega tarde, CUANDO se procesa, ENTONCES no duplica evidencia, conserva su `occurredAt` y no se asigna a una semana ni se agrega mientras PA-05 o PA-06 sigan abiertas.

**Reglas de negocio relacionadas:** RN-018, RN-026, RN-027, RN-042, RN-079, RN-108, RN-118, RN-119, RN-120, RN-122, RN-237, RN-243, RN-261  
**Notas:** Trazabilidad de alcance: M-09 y M-10. Se conserva la instrumentación disponible, pero no se implementan definiciones de KPI que permanezcan abiertas. La consulta es de solo lectura y no altera estados terminales.

## Verificación INVEST

| HU | Independent | Negotiable | Valuable | Estimable | Small | Testable | Resultado |
|---|---|---|---|---|---|---|---|
| HU-001 | Sí; delimita acceso y pertenencia | Sí; el mecanismo técnico queda para seguridad/ADR | Sí; protege el recorrido propio | Sí; entradas y resultados están acotados | Sí; cubre autenticación, no registro | Sí; credenciales y sesión tienen resultados observables | Cumple |
| HU-002 | Sí; trata selección y vigencia de ofertas | Sí; la presentación visual no se prescribe | Sí; habilita el recorrido preferencial correcto | Sí; usa identidad y vencimiento concretos | Sí; no incluye simulación financiera | Sí; selección, UTC y pertenencia son verificables | Cumple |
| HU-003 | Sí; encapsula la solicitud regular | Sí; el proveedor técnico de preevaluación queda abierto | Sí; determina si el cliente puede avanzar | Sí; estados y vigencia están definidos | Sí; no realiza la simulación | Sí; cada resultado tiene una transición esperada | Cumple |
| HU-004 | Sí; encapsula el cálculo financiero | Sí; la implementación interna puede refinarse sin cambiar reglas | Sí; produce condiciones confiables | Sí; rangos, fórmulas y errores están definidos | Sí; termina al publicar o fallar la simulación | Sí; admite vectores numéricos deterministas | Cumple |
| HU-005 | Sí; separa presentación y aceptación de contratación | Sí; el diseño de interacción es negociable | Sí; garantiza transparencia y consentimiento | Sí; versión y vigencia están definidas | Sí; no registra contrato | Sí; campos, tiempo e idempotencia son observables | Cumple |
| HU-006 | Sí; cubre intento y reserva previos | Sí; el bloqueo físico se define en ADR | Sí; protege el saldo y evita intentos duplicados | Sí; guardas e idempotencia están definidas | Sí; termina antes del registro contractual | Sí; concurrencia y conflictos son reproducibles | Cumple |
| HU-007 | Sí; cubre únicamente confirmación contractual | Sí; el diseño transaccional puede detallarse técnicamente | Sí; preserva las condiciones aceptadas | Sí; conjunto atómico y fallos están delimitados | Sí; no ejecuta el desembolso | Sí; éxito, rollback y duplicidad son comprobables | Cumple |
| HU-008 | Sí; cubre el primer despacho | Sí; bloqueo o leasing durable son alternativas | Sí; evita desembolsos duplicados | Sí; estados e identidad están definidos | Sí; no incluye reconciliación prolongada | Sí; adquisición y ejecución única son observables | Cumple |
| HU-009 | Sí; comienza ante resultado inconcluso | Sí; límites técnicos se documentan sin alterar reglas | Sí; recupera el resultado sin duplicar | Sí; respuestas y transiciones están definidas | Sí; termina al resolver o escalar | Sí; timeout, consulta y reentrega son simulables | Cumple |
| HU-010 | Sí; es una capacidad de consulta separada | Sí; la interfaz y almacenamiento analítico son negociables | Sí; da continuidad y trazabilidad al cliente | Sí; recursos, estados y deduplicación están acotados | Sí; no agrega funciones administrativas | Sí; pertenencia, estado e instrumentación son verificables | Cumple |

## Control de alcance y trazabilidad

- Las 10 historias son **Must** y cada una se traza a reglas de negocio identificadas.
- No se incluyen registro autónomo, personas reales, RENIEC, scoring real, movimientos de dinero reales, productos hipotecarios/vehiculares/empresariales, cobranza, aplicaciones móviles, chatbot, notificaciones externas ni panel administrativo completo.
- Las cuatro máquinas de estados conservan identidades independientes: solicitud, simulación, contratación y desembolso.
- Los valores configurables y temporales empleados provienen del alcance aprobado; no se completan decisiones abiertas por inferencia.

## Historias Must complementarias

Las siguientes historias cubren reglas funcionales sin una HU dedicada. Se proponen para el backlog y no sustituyen ni eliminan las HU-001–HU-010.

### HU-011: Resolver una autorización adicional de preevaluación

**Prioridad:** Must  
**Como** cliente sintético con una evaluación adicional requerida, **quiero** consultar su resolución, **para** saber si puedo continuar sin eludir la preevaluación vigente.

**Criterios de aceptación:**

- [NUEVO] DADO QUE la autorización asociada a la versión activa queda `AUTORIZADA` antes de su vencimiento, CUANDO se confirma el resultado, ENTONCES la solicitud transiciona a `SOLICITUD_HABILITADA` y permite simular dentro del máximo autorizado.
- [NUEVO] DADO QUE la autorización queda `DENEGADA`, CUANDO el cliente consulta la solicitud, ENTONCES el sistema informa que no existe oferta habilitada y bloquea simulación y contratación.
- [NUEVO] DADO QUE la autorización llega exactamente al vencimiento exclusivo de la preevaluación, CUANDO se procesa, ENTONCES no habilita la solicitud ni prolonga la vigencia por inferencia.

**Reglas de negocio relacionadas:** RN-094, RN-095, RN-157, RN-158, RN-159, RN-160, RN-165, RN-204, RN-212, RN-213  
**INVEST:** Cumple; representa una capacidad independiente, valiosa, acotada y verificable, y deja negociable el mecanismo de consulta.

### HU-012: Recuperar una contratación con resultado incierto

**Prioridad:** Must  
**Como** cliente sintético con una contratación inconclusa, **quiero** que se resuelva usando evidencia durable, **para** conocer el resultado sin duplicar contrato, reserva ni desembolso.

**Criterios de aceptación:**

- [NUEVO] DADO QUE la lectura autoritativa confirma contrato, auditoría, orden y consumo completos, CUANDO se recupera el intento, ENTONCES queda `CONTRATACION_EXITOSA` y se devuelve el contrato existente.
- [NUEVO] DADO QUE ninguna lectura acredita contrato completo ni cierre sin contrato, CUANDO se consulta cada 30 segundos, ENTONCES el intento permanece `CONTRATACION_PENDIENTE`, conserva la reserva y bloquea otro intento para la oportunidad.
- [NUEVO] DADO QUE se cumplen exactamente 15 minutos sin evidencia concluyente, CUANDO termina la recuperación automática, ENTONCES el intento pasa a `CONTRATACION_EN_REVISION_MANUAL`, mantiene la reserva y registra un incidente crítico con objetivo de resolución de 4 horas.
- [NUEVO] DADO QUE transcurren 4 horas sin evidencia concluyente, CUANDO se escala el incidente, ENTONCES no se infiere éxito, rechazo ni fallo y se conserva la revisión manual.
- [NUEVO] DADO QUE una contratación está en revisión manual, CUANDO un actor con rol `OPERADOR_REVISION_CONTRACTUAL` consulta evidencia, registra una observación o resuelve el caso, ENTONCES el sistema limita la acción a la evidencia concluyente de RN-184, conserva la reserva mientras no exista dicha evidencia y audita actor, instante, motivo, evidencia, correlación, versión y resultado.
- [NUEVO] DADO QUE un actor sin el rol contractual o un operador intenta liberar la reserva, crear otro intento o inferir un resultado sin evidencia concluyente, CUANDO ejecuta la acción, ENTONCES el sistema la rechaza sin alterar el intento ni la reserva.

**Reglas de negocio relacionadas:** RN-049, RN-050, RN-089, RN-123, RN-125, RN-126, RN-127, RN-128, RN-181, RN-182, RN-183, RN-184, RN-185, RN-226, RN-229, RN-259  
**INVEST:** Cumple; separa la recuperación del registro inicial y tiene estados, tiempos y evidencias observables.

### HU-013: Reintentar una operación recuperable de forma controlada

**Prioridad:** Must  
**Como** cliente sintético afectado por un fallo recuperable, **quiero** reintentar la misma operación, **para** continuar sin crear identidades ni efectos duplicados.

**Criterios de aceptación:**

- [NUEVO] DADO QUE una solicitud está `SOLICITUD_FALLIDA_RECUPERABLE`, CUANDO se autoriza el reintento, ENTONCES reutiliza el mismo `solicitudId`, transiciona a `SOLICITUD_VALIDANDO` y audita el intento.
- [NUEVO] DADO QUE una contratación está `CONTRATACION_FALLIDA_RECUPERABLE`, CUANDO se reintenta, ENTONCES reutiliza el mismo `intentoId`, mantiene la reserva y no crea otra contratación.
- [NUEVO] DADO QUE una operación está en fallo definitivo o el reintento controlado se agotó sin evidencia concluyente, CUANDO se solicita otro reintento automático, ENTONCES el sistema lo rechaza; solo reclasifica como definitivo con evidencia concluyente y, en otro caso, conserva el resultado pendiente.

**Reglas de negocio relacionadas:** RN-031, RN-066, RN-070, RN-180, RN-217, RN-218, RN-219, RN-220, RN-221, RN-222, RN-224, RN-225, RN-227, RN-228, RN-229, RN-230  
**INVEST:** Cumple; unifica una conducta transversal pequeña y comprobable sin prescribir la política técnica de reintentos.

### HU-014: Gestionar un desembolso en revisión manual con permisos mínimos

**Prioridad:** Must  
**Como** operador de revisión de desembolsos, **quiero** consultar evidencia y registrar observaciones dentro de mis facultades, **para** resolver casos pendientes sin crear otra operación ni alterar estados terminales.

**Criterios de aceptación:**

- [NUEVO] DADO QUE un actor tiene rol `OPERADOR_REVISION_DESEMBOLSO` y el caso está en revisión, CUANDO consulta la evidencia o solicita otra consulta de la operación original, ENTONCES el sistema permite la acción y audita identidad, instante, motivo, evidencia, correlación, versión y resultado.
- [NUEVO] DADO QUE el operador intenta crear otra operación, modificar un terminal o declarar un resultado sin evidencia concluyente, CUANDO envía la acción, ENTONCES el sistema la rechaza sin alterar el desembolso.
- [NUEVO] DADO QUE un actor sin el rol requerido, incluido un `AUDITOR`, intenta producir una transición, CUANDO solicita la acción, ENTONCES el sistema deniega el permiso; el auditor solo puede consultar registros autorizados.

**Reglas de negocio relacionadas:** RN-065, RN-072, RN-150, RN-152, RN-232, RN-233, RN-234, RN-235, RN-236, RN-238, RN-239, RN-240  
**INVEST:** Cumple; delimita una función operativa independiente con permisos y resultados verificables.

### HU-015: Verificar las puertas de calidad y seguridad antes de integrar

**Prioridad:** Must  
**Como** responsable humano de integración, **quiero** revisar evidencia de calidad y seguridad, **para** impedir la integración de cambios que incumplan la constitución.

**Criterios de aceptación:**

- [NUEVO] DADO QUE backend y frontend alcanzan por separado al menos 80 % de cobertura de líneas y existe evidencia SAST y ASVS 5.0.0 nivel 1, CUANDO se evalúa el cambio, ENTONCES puede continuar si no existen otras puertas incumplidas.
- [NUEVO] DADO QUE existe al menos un hallazgo crítico o alto abierto, CUANDO se intenta aprobar la integración, ENTONCES el sistema o proceso de control la bloquea aunque la cobertura sea exactamente 80 %.
- [NUEVO] DADO QUE existe un riesgo medio o un control declarado no aplicable, CUANDO se evalúa la puerta, ENTONCES exige aprobación humana documentada o justificación aprobada, respectivamente, antes de permitir la integración.

**Reglas de negocio relacionadas:** RN-244, RN-245, RN-246, RN-247; PRINCIPIO-011, PRINCIPIO-012, PRINCIPIO-013  
**INVEST:** Cumple como historia habilitadora; es valiosa y verificable, aunque conviene implementarla como control de entrega separado de las HUs de producto.

## Matriz canónica de trazabilidad normativa

Esta sección es la fuente canónica para la relación entre las reglas Must de `03-reglas-de-negocio.md` y las historias. Las listas incluidas dentro de cada HU son resúmenes de lectura. En esta matriz, un rango `RN-xxx–yyy` es inclusivo y cada RN tiene exactamente una HU primaria responsable. Las HUs secundarias indican colaboración, no duplican la responsabilidad.

| Reglas | HU primaria | HUs secundarias | Componentes principales | Tareas | Evidencia mínima |
|---|---|---|---|---|---|
| RN-001–002 | HU-001 | HU-002 | Identity & Access | T-005, T-021 | Integración de sesión y autorización horizontal |
| RN-003–004 | HU-002 | HU-005, HU-007 | Opportunity & Application | T-006, T-010, T-013 | Límites UTC y rechazo de oportunidad vencida |
| RN-005–006 | HU-004 | HU-005 | Financial Simulation; Web Client | T-008, T-019 | Arquitectura del módulo y ausencia de recálculo frontend |
| RN-007–012 | HU-005 | HU-004 | Web Client; Financial Simulation | T-010, T-011, T-019 | Presentación de la versión íntegra |
| RN-013 | HU-004 | HU-005 | Financial Simulation | T-010 | Nueva versión ante cambio financiero |
| RN-014 | HU-005 | — | Acceptance & Contracting | T-011 | Nueva aceptación ante cambio de versión |
| RN-015–018 | HU-007 | HU-006 | Acceptance & Contracting; Audit & Observability | T-013, T-022 | Atomicidad, unicidad y persistencia del resultado |
| RN-019–020 | HU-008 | HU-009 | Disbursement & Recovery | T-015, T-016, T-022 | Duplicidad y éxito inequívoco |
| RN-021–025 | HU-009 | HU-014 | Disbursement & Recovery | T-017, T-025 | Ambigüedad, consulta original y escalamiento |
| RN-026–027 | HU-010 | — | Audit & Observability | T-020, T-032 | Deduplicación, idempotencia y evidencia de atribución KPI |
| RN-028 | HU-015 | HU-001–HU-010 | Web Client | T-019, T-026 | Operación por teclado |
| RN-029–030 | HU-005 | HU-006 | Acceptance & Contracting | T-011, T-012, T-022 | Aceptación propia y concurrencia |
| RN-031 | HU-013 | — | Módulos de dominio | T-024, T-031 | Reintento solo desde estado recuperable |
| RN-032–033 | HU-009 | HU-014 | Disbursement & Recovery; Audit & Observability | T-017, T-025, T-031 | Terminal inmutable y ajuste separado |
| RN-034 | HU-006 | — | Acceptance & Contracting | T-012 | Identidad y estado del intento |
| RN-035 | HU-007 | HU-006 | Acceptance & Contracting | T-013, T-022 | Contrato único entre intentos |
| RN-036–041 | HU-004 | HU-005 | Financial Simulation | T-008, T-009 | Vectores numéricos deterministas |
| RN-042 | HU-008 | HU-009 | Disbursement & Recovery | T-015, T-016 | `operationId` único y persistente |
| RN-043–044 | HU-006 | — | Acceptance & Contracting | T-012, T-021 | Idempotencia y guardas con clave nueva |
| RN-045 | HU-012 | HU-010 | Acceptance & Contracting | T-014 | Recuperación del contrato existente |
| RN-046–047 | HU-007 | HU-012 | Acceptance & Contracting; Audit & Observability | T-013, T-022 | Rollback y fallo separado |
| RN-048 | HU-006 | HU-012 | Acceptance & Contracting | T-012 | Bloqueo por contrato o intento incierto |
| RN-049–050 | HU-012 | — | Acceptance & Contracting | T-014 | Pendencia y resolución por evidencia |
| RN-051–055 | HU-009 | HU-013 | Disbursement & Recovery | T-017, T-022, T-024 | Reconciliación y concurrencia |
| RN-056 | HU-007 | HU-005 | Acceptance & Contracting | T-013 | Revalidación de vigencia contractual |
| RN-057–059 | HU-005 | HU-004 | Financial Simulation; Acceptance & Contracting | T-010, T-011 | Vigencia e independencia de simulaciones |
| RN-060–064 | HU-004 | — | Financial Simulation | T-008–T-010 | Límites, cronograma, TEA cero y TCEA inválida |
| RN-065 | HU-001 | HU-012, HU-014 | Identity & Access | T-005, T-025, T-030 | Asignación y restricción de roles |
| RN-066 | HU-013 | HU-003, HU-004, HU-006 | Módulos de dominio; Audit & Observability | T-024 | Clasificación y causa de error |
| RN-067–068 | HU-006 | — | Acceptance & Contracting | T-012, T-021 | Conflicto y retención idempotente |
| RN-069–070 | HU-013 | HU-003–HU-009 | Módulos de dominio; Audit & Observability | T-020, T-024, T-031 | Estados canónicos y evidencia terminal |
| RN-071 | HU-006 | HU-004, HU-005, HU-007 | Acceptance & Contracting | T-012, T-022 | Serialización y orden de bloqueo |
| RN-072 | HU-014 | HU-009 | Disbursement & Recovery | T-025 | Pendencia durante revisión manual |
| RN-073 | HU-004 | — | Financial Simulation | T-008, T-009 | `BigDecimal`, precisión y períodos uniformes |
| RN-074 | HU-002 | HU-003–HU-005 | Opportunity & Application | T-006, T-007, T-010 | Vencimiento exclusivo UTC |
| RN-075–077 | HU-005 | HU-004 | Web Client; Financial Simulation | T-010, T-019 | Seguro, pago y costo total visibles |
| RN-078–079 | HU-007 | HU-006, HU-012 | Acceptance & Contracting; Audit & Observability | T-013, T-014 | Vigencias y correlación del intento |
| RN-080 | HU-015 | HU-001–HU-010 | Web Client | T-019, T-026 | Etiquetas identificables |
| RN-081–082 | HU-007 | HU-003–HU-009 | Audit & Observability | T-020, T-022, T-031 | Auditoría y control de versión de transición |
| RN-083–084 | HU-004 | HU-005 | Financial Simulation | T-008–T-010 | Publicación válida y costo total |
| RN-085 | HU-007 | HU-006 | Acceptance & Contracting | T-013, T-022 | Reloj autoritativo bajo bloqueos |
| RN-086–089 | HU-006 | HU-012 | Acceptance & Contracting | T-012, T-014, T-022 | Línea compartida, reserva y pendencia |
| RN-090–091 | HU-007 | HU-006 | Acceptance & Contracting | T-013, T-022 | Conversión o liberación única |
| RN-092 | HU-005 | — | Acceptance & Contracting | T-011, T-021 | Aceptación idempotente |
| RN-093 | HU-003 | HU-004 | Opportunity & Application | T-007, T-010 | Oferta base habilitada para simular |
| RN-094–095 | HU-011 | HU-003 | Opportunity & Application | T-023 | Evaluación adicional informada y bloqueada |
| RN-096 | HU-007 | — | Acceptance & Contracting | T-013, T-022 | Rechazo en límite de vigencia |
| RN-097–098 | HU-004 | HU-003 | Financial Simulation | T-008, T-010 | Máximo de preevaluación y TEA pizarra |
| RN-099 | HU-007 | HU-008 | Acceptance & Contracting | T-013, T-022 | Orden única dentro de transacción |
| RN-100 | HU-008 | HU-009 | Disbursement & Recovery | T-016, T-022 | Recuperación de adquisición no confirmada |
| RN-101–104 | HU-009 | HU-014 | Disbursement & Recovery; Audit & Observability | T-017, T-025 | Consulta, conservación y ajuste auditable |
| RN-105 | HU-005 | — | Acceptance & Contracting | T-011 | Evidencia de aceptación expresa |
| RN-106 | HU-007 | HU-005 | Acceptance & Contracting | T-013 | Uso exacto de la aceptación |
| RN-107–108 | HU-006 | HU-012 | Acceptance & Contracting | T-012, T-014 | Intento-reserva y bloqueo de repetición |
| RN-109–112 | HU-003 | — | Opportunity & Application | T-007 | Indisponibilidad y ausencia de oferta |
| RN-113–114 | HU-005 | HU-007, HU-011 | Acceptance & Contracting | T-011, T-013 | Primer vencimiento e invalidación |
| RN-115–117 | HU-006 | — | Acceptance & Contracting | T-012, T-021 | Idempotencia contractual completa |
| RN-118–120 | HU-010 | HU-001 | Web Client; Audit & Observability | T-018, T-021 | Consulta propia y titularidad |
| RN-121–122 | HU-001 | HU-002–HU-010 | Identity & Access | T-005, T-021 | Cuenta sintética y pertenencia |
| RN-123 | HU-012 | HU-006 | Acceptance & Contracting | T-014 | Recuperación por identidad persistida |
| RN-124 | HU-001 | — | Identity & Access | T-005 | Contraseña independiente del DNI |
| RN-125–128 | HU-012 | HU-007 | Acceptance & Contracting | T-014, T-022 | Evidencia durable, éxito, cierre o pendencia |
| RN-129–132 | HU-008 | HU-009 | Disbursement & Recovery | T-016, T-022 | Orden durable, adquisición y primera entrega |
| RN-133–134 | HU-009 | HU-014 | Disbursement & Recovery | T-017, T-025 | Consulta/reentrega y escalamiento |
| RN-135 | HU-008 | HU-009 | Disbursement & Recovery | T-015, T-022 | Deduplicación atómica del core |
| RN-136–139 | HU-006 | HU-007 | Acceptance & Contracting | T-012, T-022 | Orden de bloqueos y revalidación |
| RN-140–141 | HU-005 | HU-006 | Acceptance & Contracting | T-011, T-022 | Aceptación concurrente revalidada |
| RN-142–143 | HU-006 | HU-007 | Acceptance & Contracting | T-012, T-022 | Competencia por saldo y transición única |
| RN-144 | HU-009 | — | Disbursement & Recovery | T-017 | Respuesta no terminal pendiente |
| RN-145–147 | HU-008 | HU-009 | Disbursement & Recovery | T-015–T-017, T-022 | Persistencia de terminales del core |
| RN-148–149 | HU-009 | — | Disbursement & Recovery | T-017, T-022 | `NO_ENCONTRADA` y errores inconclusos |
| RN-150 | HU-014 | HU-009 | Disbursement & Recovery | T-025 | Rechazo descubierto en revisión |
| RN-151 | HU-008 | HU-009 | Disbursement & Recovery | T-017, T-022 | Respuesta tardía sobre estado pendiente |
| RN-152 | HU-014 | HU-009 | Disbursement & Recovery; Audit & Observability | T-025 | Contradicción terminal separada |
| RN-153 | HU-008 | — | Disbursement & Recovery | T-015 | Terminal inmutable del core |
| RN-154–156 | HU-003 | — | Opportunity & Application | T-007 | Identidad, vigencia y oferta base |
| RN-157–160 | HU-011 | HU-003 | Opportunity & Application | T-023 | Evaluación y autorización adicional |
| RN-161–164 | HU-003 | HU-011 | Opportunity & Application | T-007, T-023 | Sin oferta, indisponibilidad y versionado |
| RN-165 | HU-011 | HU-003 | Audit & Observability | T-023 | Auditoría de preevaluación/autorización |
| RN-166 | HU-003 | HU-004, HU-005 | Opportunity & Application | T-007, T-010, T-011 | Reemplazo e invalidación auditada |
| RN-167–170 | HU-002 | HU-006 | Opportunity & Application | T-006, T-012 | Identidad y selección de oferta preferencial |
| RN-171–177 | HU-003 | HU-002 | Opportunity & Application | T-007 | Inicio explícito, unicidad y cierre de solicitud |
| RN-178–179 | HU-006 | HU-007 | Acceptance & Contracting | T-012, T-013, T-022 | Rechazo/fallo definitivo y liberación |
| RN-180 | HU-013 | HU-006 | Acceptance & Contracting | T-024 | Fallo recuperable con misma reserva |
| RN-181–185 | HU-012 | HU-006 | Acceptance & Contracting; Audit & Observability | T-014 | Frecuencia, escalamiento y objetivo de cuatro horas |
| RN-186–201 | HU-004 | HU-005, HU-006 | Financial Simulation | T-008–T-010 | Vectores, configuración y datos obligatorios |
| RN-202 | HU-006 | HU-004 | Acceptance & Contracting | T-012 | Revalidación de línea al reservar |
| RN-203 | HU-003 | HU-011 | Opportunity & Application | T-007, T-023 | Vigencia de oferta base |
| RN-204 | HU-011 | HU-003 | Opportunity & Application | T-023 | Autorización vigente obligatoria |
| RN-205–206 | HU-003 | HU-005, HU-011 | Opportunity & Application | T-007, T-011, T-023 | Reconsulta y reemplazo de versión |
| RN-207 | HU-004 | HU-005 | Financial Simulation | T-010 | Configuración nueva solo a simulaciones nuevas |
| RN-208 | HU-006 | HU-004 | Acceptance & Contracting | T-012 | Cambio de línea aplicado al reservar |
| RN-209 | HU-005 | HU-002, HU-004 | Financial Simulation; Acceptance & Contracting | T-010, T-011 | Oferta reemplazada e invalidación |
| RN-210–211 | HU-003 | — | Opportunity & Application | T-007 | Preevaluación y habilitación de solicitud |
| RN-212–213 | HU-011 | HU-003 | Opportunity & Application | T-023 | Espera y autorización de solicitud |
| RN-214 | HU-011 | HU-003 | Opportunity & Application | T-023 | Denegación de autorización |
| RN-215–216 | HU-003 | — | Opportunity & Application | T-007 | Pendencia y guarda para simular |
| RN-217–222 | HU-013 | HU-003, HU-004 | Módulos de solicitud y simulación | T-024, T-031 | Recuperación/reintento con misma identidad |
| RN-223 | HU-007 | HU-006 | Acceptance & Contracting | T-013, T-022 | Vencimiento contractual y rollback |
| RN-224–230 | HU-013 | HU-006, HU-012 | Módulos de dominio; Audit & Observability | T-024, T-031 | Clasificación, reintento, pendencia y escalamiento |
| RN-231 | HU-009 | HU-014 | Disbursement & Recovery | T-017, T-025 | Entrada a revisión manual |
| RN-232–236 | HU-014 | HU-009 | Identity & Access; Disbursement & Recovery; Audit & Observability | T-025, T-030 | Permisos, evidencia, auditoría y conflicto |
| RN-237 | HU-001 | HU-002–HU-010 | Identity & Access | T-005, T-021 | Recursos propios del cliente sintético |
| RN-238–239 | HU-014 | HU-015 | Identity & Access; Audit & Observability | T-025, T-026 | Facultades de operador y auditor |
| RN-240 | HU-008 | HU-009 | Identity & Access; Disbursement & Recovery | T-015–T-017 | Facultades del servicio interno |
| RN-241–243 | HU-001 | HU-005–HU-010 | Identity & Access | T-005, T-021 | Sesión, bloqueo temporal y mínima revelación |
| RN-244–247 | HU-015 | — | Audit & Observability; CI | T-026, T-030 | ASVS, SAST y decisiones humanas documentadas |
| RN-248–252 | HU-006 | HU-010 | Acceptance & Contracting; web-api | T-004, T-012, T-021 | Idempotencia aislada y contrato técnico |
| RN-253–258 | HU-005 | HU-004, HU-015 | Web Client; Acceptance & Contracting; web-api | T-004, T-011, T-019, T-021 | Presentación, integridad, aceptación y deduplicación |
| RN-259 | HU-012 | HU-006, HU-015 | Identity & Access; Acceptance & Contracting; Audit & Observability | T-014, T-030 | Revisión contractual con mínimo privilegio y evidencia concluyente |
| RN-260 | HU-003 | HU-013 | Opportunity & Application; Audit & Observability | T-004, T-007, T-020, T-031 | Cancelación propia, idempotente, serializada, auditable y sin entidades dependientes |
| RN-261 | HU-010 | HU-002, HU-004, HU-005, HU-007 | Audit & Observability; Web Client; módulos backend | T-003, T-004, T-020, T-032 | Catálogo canónico, identidad, UTC, correlación, fuente, esquema y deduplicación sin agregación no aprobada |

La cobertura de una regla se considera completa únicamente cuando existen la tarea y la evidencia mínima indicadas. Una referencia documental por sí sola no prueba que la regla haya sido implementada o verificada.
