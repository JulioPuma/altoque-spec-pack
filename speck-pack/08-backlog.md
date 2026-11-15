# Altoque — Backlog técnico inicial

**Fuente:** `02-alcance.md`, `06-user-stories.md` y `07-architecture.md`  
**Criterio de orden:** prioridad y, dentro de ella, dependencias.  
**Nota de ejecución:** cada tarea está acotada para ser completada por un agente en menos de dos horas. Las verificaciones que requieren ejecutar código deben ser realizadas por una persona responsable o por CI autorizado, conforme a la restricción operativa de la arquitectura.

**Trazabilidad normativa:** la matriz canónica RN → HU → componente → tarea → evidencia se mantiene en `06-user-stories.md`.  

## Tareas Must

### T-000: Registrar la aprobación de arquitectura base

**HU relacionada:** No aplica; tarea habilitadora  
**Requisitos relacionados:** PRINCIPIO-002, PRINCIPIO-006, PRINCIPIO-008; ADR-001, ADR-002, ADR-003  
**Tipo:** Enabler  
**Componente:** Arquitectura transversal  
**Prioridad:** Must  
**Tamaño:** S (< 30 min)  
**Dependencias:** Ninguna  
**Descripción:** Conservar la aprobación humana de ADR-001, ADR-002, ADR-003 y PostgreSQL como persistencia oficial para la primera evaluación. La aprobación no fija versiones del stack ni completa los parámetros pendientes de la especificación de seguridad. Cualquier sustitución posterior de una tecnología principal requiere un nuevo ADR y aprobación humana.  
**Criterio de completitud:** Sí si los tres ADR y PostgreSQL constan como aceptados para la primera evaluación, con alcance y fecha de aprobación, y las decisiones pendientes permanecen identificadas; no en caso contrario.

### T-001: Fijar el stack y crear la estructura modular

**HU relacionada:** No aplica; tarea habilitadora  
**Requisitos relacionados:** PRINCIPIO-001, PRINCIPIO-002; ADR-001  
**Tipo:** Enabler  
**Componente:** Web Client; módulos backend  
**Prioridad:** Must  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-000  
**Descripción:** Registrar las versiones aprobadas de JDK, Spring Boot, Angular, TypeScript, PostgreSQL, drivers y herramientas de construcción. Crear los módulos `web-api`, `identity-access`, `opportunity-application`, `financial-domain`, `contracting`, `disbursement` y `audit`, respetando las dependencias descritas en la arquitectura.  
**Criterio de completitud:** Sí si los manifiestos fijan versiones exactas aprobadas y la estructura no introduce dependencias de dominio prohibidas; no en caso contrario.

### T-002: Crear el esquema base y migraciones de identidad y oportunidades

**HU relacionada:** HU-001, HU-002, HU-003  
**Componente:** Identity & Access; Opportunity & Application  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-001  
**Descripción:** Crear migraciones versionadas para `Customer`, `CreditLine`, `PreferredOffer`, `LoanApplication` y `PreEvaluation`, usando UUID, `timestamptz`, `numeric` y columna `version`. Incluir restricciones de PEN, límite de línea y una sola solicitud regular activa por cliente.  
**Criterio de completitud:** Sí si la migración crea las cinco entidades y todas sus restricciones aprobadas sin usar `float`/`double`; no en caso contrario.

### T-003: Crear el esquema de simulación, contratación, desembolso y auditoría

**HU relacionada:** HU-004, HU-005, HU-006, HU-007, HU-008, HU-009, HU-010  
**Componente:** Financial Simulation; Acceptance & Contracting; Disbursement & Recovery; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-001, T-002  
**Descripción:** Crear migraciones para `Simulation`, `Installment`, `Acceptance`, `ContractAttempt`, `CreditReservation`, `LoanContract`, `DisbursementOrder`, `DisbursementEvidence`, `StateTransition`, `IdempotencyRecord` e `InteractionEvent`. Añadir claves únicas, relaciones, estados/versiones y restricciones de una reserva por intento, un contrato por oportunidad y una orden/operación por contrato. Para `InteractionEvent`, incluir los campos, esquemas y claves de deduplicación de RN-261.  
**Criterio de completitud:** Sí si las once entidades y las restricciones de unicidad descritas en arquitectura existen en migraciones versionadas y los eventos KPI no admiten tipos, campos obligatorios o duplicados contrarios a RN-261; no en caso contrario.

### T-004: Publicar el contrato OpenAPI base

**HU relacionada:** HU-001–HU-015  
**Componente:** Web Client; web-api  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-001, T-030  
**Descripción:** Documentar en OpenAPI los endpoints `/auth/sessions`, `/opportunities`, `/applications`, `/simulations`, aceptaciones, intentos, consultas de operaciones, captura de interacciones KPI y revisión manual contractual y de desembolso. Incluir DTO, estados HTTP, error uniforme, cadenas decimales, `Idempotency-Key`, CSRF, `X-Correlation-Id`, roles autorizados, esquema de RN-261 y mínima revelación.  
**Criterio de completitud:** Sí si el contrato valida y representa todos los endpoints y catálogos definidos en arquitectura; no en caso contrario.

### T-005: Implementar autenticación de cuentas sintéticas y sesión segura

**HU relacionada:** HU-001  
**Componente:** Identity & Access  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-002, T-004  
**Descripción:** Implementar autenticación contra cuentas precargadas con hash aprobado, respuesta genérica ante fallo y rotación de sesión. Emitir cookie `HttpOnly`, `Secure`, `SameSite=Lax`, exigir CSRF en comandos y derivar `customerId` exclusivamente de la sesión.  
**Criterio de completitud:** Sí si una cuenta activa inicia sesión y credenciales inválidas o cuentas no habilitadas no crean sesión ni revelan el dato fallido; no en caso contrario.

### T-006: Implementar consulta de oportunidades propias

**HU relacionada:** HU-002  
**Componente:** Opportunity & Application  
**Prioridad:** Must  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-002, T-004, T-005  
**Descripción:** Implementar `GET /opportunities` filtrando por el cliente de la sesión y calculando la vigencia exclusiva de 30 días en UTC. Devolver lista vacía cuando corresponda y ocultar recursos ajenos como inexistentes.  
**Criterio de completitud:** Sí si la respuesta contiene solo ofertas propias vigentes y consultar una oferta ajena no revela su existencia; no en caso contrario.

### T-007: Implementar inicio y resolución de solicitud regular

**HU relacionada:** HU-003  
**Componente:** Opportunity & Application  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-002, T-004, T-005  
**Descripción:** Implementar la creación idempotente de una única solicitud activa, su transición a validación y el adaptador de preevaluación. Permitir el inicio explícito a cualquier cliente sintético, tenga o no una oferta preferencial vigente, sin iniciar el recorrido automáticamente ni invalidar la oferta. Persistir resultados versionados y resolver oferta base, evaluación adicional, sin oferta y no disponible, con vencimiento exclusivo de 24 horas. Implementar además la cancelación idempotente por el propietario desde `SOLICITUD_INICIADA` o `SOLICITUD_HABILITADA`, serializando por oportunidad y rechazándola si existe cualquier simulación, intento, reserva o contrato.  
**Criterio de completitud:** Sí si cada resultado produce el estado definido, una repetición no crea otro `applicationId`, un cliente con oferta preferencial puede iniciar explícitamente su solicitud regular, ambas oportunidades permanecen independientes sobre la línea compartida y las pruebas de cancelación demuestran autorización, estados permitidos, idempotencia, auditoría, concurrencia con simulación, rechazo ante entidades dependientes y nueva identidad posterior; no en caso contrario.

### T-008: Implementar el cálculo financiero determinista

**HU relacionada:** HU-004  
**Componente:** Financial Simulation  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-001  
**Descripción:** Implementar en `financial-domain` tasa mensual, cuota francesa, cronograma, ajuste de última cuota, seguro, costo total y TCEA por bisección. Usar `BigDecimal`, precisión 34, `HALF_UP`, `Clock` inyectado y las reglas exactas de fechas del alcance.  
**Criterio de completitud:** Sí si el módulo produce todos los campos sin `float`/`double` ni dependencia de web, persistencia o reloj del sistema; no en caso contrario.

### T-009: Probar los vectores financieros Must

**HU relacionada:** HU-004  
**Componente:** Financial Simulation  
**Prioridad:** Must  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-008  
**Descripción:** Crear pruebas unitarias para TEA cero con una cuota, ajuste final a tres cuotas, ancla día 31, año bisiesto y TIR sin solución. Verificar importes, fechas, saldo cero, TCEA presentada y fallo explícito.  
**Criterio de completitud:** Sí si existen pruebas automatizadas con aserciones exactas para los cinco vectores normativos; no en caso contrario.

### T-010: Implementar creación y persistencia de simulaciones

**HU relacionada:** HU-004  
**Componente:** Financial Simulation  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-003, T-004, T-006, T-007, T-008  
**Descripción:** Implementar `POST /simulations` para oferta preferencial o solicitud habilitada. Validar monto, incremento, plazo, máximo, saldo y datos obligatorios; congelar configuración y persistir simulación, cronograma y hash inmutable con vigencia de 30 minutos.  
**Criterio de completitud:** Sí si solo una simulación completa y almacenada se publica como `SIMULACION_EXITOSA` y toda entrada inválida queda rechazada sin valores por defecto; no en caso contrario.

### T-011: Implementar aceptación expresa e idempotente

**HU relacionada:** HU-005  
**Componente:** Acceptance & Contracting  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-003, T-004, T-010  
**Descripción:** Implementar el endpoint de aceptación verificando pertenencia, integridad, estado, versión y vigencia exclusiva. Persistir la acción expresa y la evidencia de condiciones; devolver el registro original ante una repetición equivalente.  
**Criterio de completitud:** Sí si una aceptación válida conserva la versión mostrada y las repeticiones crean como máximo un registro; no en caso contrario.

### T-012: Implementar intento contractual y reserva atómica

**HU relacionada:** HU-006  
**Componente:** Acceptance & Contracting  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-003, T-004, T-011  
**Descripción:** Implementar la preparación contractual bloqueando primero oportunidad y luego línea, y revalidando guardas después de cada espera. En una transacción, persistir `ContractAttempt`, `IdempotencyRecord` y reserva activa; una misma clave y hash devuelve el intento, y un hash distinto produce conflicto.  
**Criterio de completitud:** Sí si intento y reserva se confirman juntos, el saldo nunca se sobrerreserva y la idempotencia no puede evadir las guardas; no en caso contrario.

### T-013: Implementar confirmación contractual atómica

**HU relacionada:** HU-007  
**Componente:** Acceptance & Contracting; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-012  
**Descripción:** Revalidar oportunidad, línea, aceptación y vigencias bajo el orden de bloqueo aprobado. Confirmar en una transacción contrato, transición auditable, orden única de desembolso, conversión de reserva a consumo y estado exitoso del intento.  
**Criterio de completitud:** Sí si el fallo de cualquier escritura revierte el conjunto completo y una oportunidad nunca obtiene dos contratos; no en caso contrario.

### T-014: Implementar recuperación de contratación incierta

**HU relacionada:** HU-012  
**Componente:** Identity & Access; Acceptance & Contracting; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-005, T-013, T-030  
**Descripción:** Consultar evidencia durable del intento cada 30 segundos hasta 15 minutos sin liberar la reserva por tiempo transcurrido. Confirmar éxito solo ante el conjunto contractual completo, cerrar sin contrato solo con evidencia concluyente y escalar el resto a `CONTRATACION_EN_REVISION_MANUAL`. Restringir la revisión al rol `OPERADOR_REVISION_CONTRACTUAL`, conservar la reserva y el bloqueo de otro intento, auditar toda acción y aplicar el objetivo operativo de resolución de cuatro horas sin inferir resultados por vencimiento. Publicar y mantener un runbook de consulta, resolución, escalamiento y contradicción contractual.  
**Evidencia requerida:** pruebas automatizadas de lectura inconclusa, conjunto contractual completo, rollback durable, ingreso a revisión al minuto 15, incumplimiento del objetivo de cuatro horas, autorización por rol, auditoría y conservación de reserva; runbook aprobado de revisión contractual.  
**Criterio de completitud:** Sí si ningún resultado ambiguo se convierte en éxito o fallo, solo evidencia de RN-184 resuelve el caso, todo acceso o decisión manual queda auditado, la reserva permanece mientras no existe cierre concluyente y el runbook identifica responsables y escalamiento; no en caso contrario.

### T-015: Implementar el core de desembolso simulado idempotente

**HU relacionada:** HU-008  
**Componente:** Disbursement & Recovery  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-003, T-004  
**Descripción:** Implementar `POST` y `GET /internal/v1/disbursements` detrás de autenticación de servicio. Deduplicar por `operationId` y hash de payload: una repetición idéntica devuelve la operación original y datos distintos producen conflicto sin ejecución.  
**Criterio de completitud:** Sí si un `operationId` puede producir como máximo una operación lógica y el contrato de consulta devuelve el catálogo aprobado; no en caso contrario.

### T-016: Implementar adquisición y despacho exclusivo de órdenes

**HU relacionada:** HU-008  
**Componente:** Disbursement & Recovery  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-013, T-015  
**Descripción:** Crear el worker que adquiere exclusivamente una orden `DESEMBOLSO_SOLICITADO`, confirma durablemente `DESEMBOLSO_PROCESANDO` y recién entonces invoca el core con el payload persistido. Usar lease recuperable, versión y semántica equivalente a `FOR UPDATE SKIP LOCKED`.  
**Criterio de completitud:** Sí si dos workers concurrentes no pueden iniciar dos ejecuciones lógicas de la misma orden; no en caso contrario.

### T-017: Implementar recuperación y revisión de desembolso

**HU relacionada:** HU-009  
**Componente:** Disbursement & Recovery; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-016  
**Descripción:** Ante timeout, desconexión, respuesta inválida, estado desconocido o `NOT_FOUND`, conservar pendiente y consultar el `operationId` original. Permitir reentrega solo del payload original, persistir evidencia append-only y escalar sin alterar terminales contradictorios.  
**Criterio de completitud:** Sí si la recuperación nunca crea otro identificador ni sobrescribe un estado terminal confirmado; no en caso contrario.

### T-018: Implementar consultas posteriores de intentos y operaciones

**HU relacionada:** HU-010  
**Componente:** Web Client; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-005, T-014, T-017  
**Descripción:** Implementar las lecturas puras de intento y listado paginado de operaciones con cursor opaco y límite 1–50. Filtrar por sesión, devolver solo estado/version durable y no iniciar transiciones durante la consulta.  
**Criterio de completitud:** Sí si el cliente ve sus operaciones pendientes y resueltas y un identificador ajeno recibe la misma respuesta que uno inexistente; no en caso contrario.

### T-019: Implementar el recorrido Angular accesible y resiliente

**HU relacionada:** HU-001–HU-010  
**Componente:** Web Client  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-004, T-005, T-006, T-007, T-010, T-011, T-012, T-018  
**Descripción:** Construir rutas y formularios para autenticación, oportunidad, solicitud, simulación, aceptación, contratación y consulta. Renderizar los cálculos del backend sin recalcularlos; conservar clave y cuerpo ante reintento, usar backoff solo en lecturas y asegurar teclado, etiquetas, foco y diseño angosto.  
**Criterio de completitud:** Sí si el recorrido principal consume los contratos publicados, no usa `localStorage` como autoridad y es operable por teclado; no en caso contrario.

### T-020: Instrumentar auditoría, correlación y métricas técnicas

**HU relacionada:** HU-005, HU-006, HU-007, HU-008, HU-009, HU-010  
**Componente:** Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-003, T-013, T-017, T-018  
**Descripción:** Registrar transiciones append-only, los cinco eventos canónicos de RN-261 y logs JSON con `correlationId`, enmascarando secretos y credenciales. Derivar identidad y pertenencia en backend, validar esquema, fuente, versión e instante UTC, deduplicar la primera visualización por cliente y oportunidad y conservar otras interacciones con `interactionId`. Exponer métricas técnicas de error, latencia, conflictos, lag, órdenes pendientes y revisiones, sin inventar fórmulas KPI abiertas.  
**Criterio de completitud:** Sí si cada transición relevante es trazable, los eventos cumplen RN-261, una repetición técnica no duplica evidencia, un evento tardío no se reasigna ni agrega y ninguna salida registra contraseñas, cookies, CSRF o cabeceras de autorización; no en caso contrario.

### T-021: Probar autorización horizontal e idempotencia Must

**HU relacionada:** HU-001, HU-005, HU-006, HU-010  
**Componente:** Identity & Access; Acceptance & Contracting  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-005, T-011, T-012, T-018  
**Descripción:** Crear pruebas de integración para recursos ajenos, sesión vencida, aceptación repetida, misma clave/cuerpo y misma clave con cuerpo distinto. Verificar mínima revelación, aislamiento de claves entre clientes y ausencia de efectos duplicados.  
**Criterio de completitud:** Sí si existen pruebas automatizadas que cubren todos los casos y sus aserciones comprueban estado y cantidad de registros; no en caso contrario.

### T-022: Probar atomicidad, concurrencia y recuperación Must

**HU relacionada:** HU-006, HU-007, HU-008, HU-009  
**Componente:** Acceptance & Contracting; Disbursement & Recovery  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-013, T-014, T-016, T-017  
**Descripción:** Crear pruebas de integración para dos reservas que compiten por saldo, aceptación frente a contratación, rollback de cualquiera de las cuatro escrituras contractuales y dos workers sobre una orden. Cubrir también respuesta perdida, timeout, `NOT_FOUND` y contradicción de un terminal.  
**Criterio de completitud:** Sí si las pruebas demuestran una sola reserva/contrato/operación lógica y preservación de pendientes y terminales según las reglas; no en caso contrario.

### T-023: Implementar autorización adicional de preevaluación

**HU relacionada:** HU-011  
**Componente:** Opportunity & Application; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-003, T-004, T-007  
**Descripción:** Implementar la consulta y persistencia versionada de la autorización asociada a una preevaluación en `EVALUACION_ADICIONAL_REQUERIDA`. Mantener la solicitud en `SOLICITUD_ESPERANDO_AUTORIZACION` mientras no exista decisión vigente; transicionar a `SOLICITUD_HABILITADA` ante `AUTORIZADA` y a `SOLICITUD_RECHAZADA` ante `DENEGADA`, sin reutilizar otra versión ni completar datos ausentes. Auditar identidad, versión, instante, resultado y correlación.  
**Criterio de completitud:** Sí si las pruebas automatizadas demuestran las transiciones autorizada, denegada, pendiente, vencida y versión reemplazada, y ninguna solicitud puede simular o contratar sin autorización vigente; no en caso contrario.

### T-024: Implementar reintentos controlados por máquina de estados

**HU relacionada:** HU-013  
**Componente:** Opportunity & Application; Financial Simulation; Acceptance & Contracting; Disbursement & Recovery; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-007, T-010, T-014, T-017, T-020  
**Descripción:** Implementar los reintentos controlados de solicitud, simulación y contratación desde estados `FALLIDA_RECUPERABLE`, reutilizando la identidad de la operación y auditando cada intento. Prohibir el reintento automático de fallos definitivos; conservar pendiente y escalar cuando la evidencia no permita clasificar el resultado. Para desembolso, limitar la recuperación a consulta o reentrega idempotente del comando original.  
**Criterio de completitud:** Sí si las pruebas automatizadas cubren reintento recuperable, rechazo de reintento definitivo, agotamiento con y sin evidencia concluyente, conservación de identidad y ausencia de efectos duplicados en las cuatro máquinas; no en caso contrario.

### T-025: Implementar operación manual de desembolsos en revisión

**HU relacionada:** HU-014, HU-009  
**Componente:** Identity & Access; Disbursement & Recovery; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-004, T-005, T-017, T-020, T-030  
**Descripción:** Publicar los casos en `DESEMBOLSO_EN_REVISION_MANUAL` únicamente al rol `OPERADOR_REVISION_DESEMBOLSO`; permitir consultar evidencia, registrar observaciones y solicitar otra consulta de la operación original. Impedir crear otra operación, alterar un terminal o declarar un resultado sin evidencia concluyente. Auditar actor, instante, motivo, evidencia, correlación, versión y resultado, y crear un expediente separado ante contradicción terminal.  
**Criterio de completitud:** Sí si las pruebas de autorización y transición demuestran mínimo privilegio, auditoría completa, conservación del `operationId`, resolución solo por evidencia concluyente y rechazo de toda modificación de terminal; no en caso contrario.

### T-030: Elaborar y aprobar la especificación de seguridad

**HU relacionada:** HU-001, HU-012, HU-014, HU-015  
**Requisitos relacionados:** PRINCIPIO-006, PRINCIPIO-011, PRINCIPIO-012, PRINCIPIO-013; RN-065, RN-121–124, RN-232–247, RN-259  
**Tipo:** Enabler de seguridad  
**Componente:** Identity & Access; Web Client; web-api; módulos backend  
**Prioridad:** Must  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-000  
**Descripción:** Documentar y someter a aprobación humana la política técnica de contraseñas y bloqueo temporal, ciclo de sesión, cookies, CSRF, rate limits, roles y matriz de permisos, autenticación de servicio, secretos, cifrado, retención, auditoría de accesos y threat model mínimo. Incluir expresamente `OPERADOR_REVISION_CONTRACTUAL`, `OPERADOR_REVISION_DESEMBOLSO`, `AUDITOR`, `CLIENTE_SINTETICO` y `SERVICIO_INTERNO`. No incorporar credenciales ni datos reales.  
**Criterio de completitud:** Sí si existe una especificación versionada y aprobada por una persona responsable, sin parámetros abiertos requeridos por T-004, T-005, T-014 o T-025, con amenazas, controles, responsables y decisiones de retención trazables; no en caso contrario.

### T-031: Probar exhaustivamente las cuatro máquinas de estados

**HU relacionada:** HU-003, HU-004, HU-006, HU-007, HU-008, HU-009, HU-012, HU-013, HU-014  
**Componente:** Opportunity & Application; Financial Simulation; Acceptance & Contracting; Disbursement & Recovery; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-007, T-010, T-014, T-017, T-023, T-024, T-025  
**Descripción:** Crear pruebas parametrizadas para solicitud, simulación, contratación y desembolso que recorran cada transición permitida y rechacen cada transición no permitida. Verificar prefijos canónicos, terminalidad, `FALLIDA_RECUPERABLE` frente a `FALLIDA_DEFINITIVA`, estado origen, versión esperada, auditoría y ausencia de efectos ante conflicto concurrente. Para `SOLICITUD_CANCELADA`, cubrir los dos estados origen, titularidad, idempotencia, ausencia de entidades dependientes, carrera con simulación y prohibición de reutilizar el `solicitudId`. Mantener los casos sincronizados con las matrices normativas de `03-reglas-de-negocio.md`.  
**Criterio de completitud:** Sí si cada arco permitido tiene al menos un caso positivo, cada estado tiene casos negativos representativos, ningún terminal puede mutarse, todo reintento conserva identidad y CI detecta divergencias entre catálogo, persistencia y API; no en caso contrario.

### T-032: Probar la evidencia de instrumentación KPI

**HU relacionada:** HU-010  
**Requisitos relacionados:** M-09; RN-026, RN-027  
**Componente:** Audit & Observability  
**Prioridad:** Must  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-006, T-013, T-020  
**Descripción:** Crear pruebas de contrato para los cinco eventos de RN-261. Demostrar que múltiples visualizaciones de la misma oportunidad por el mismo cliente conservan una única visualización contabilizable, que las interacciones posteriores mantienen identidad propia, que aceptación y contrato solo se emiten desde confirmaciones durables y que una repetición técnica no duplica evidencia. Verificar campos obligatorios por tipo, pertenencia, correlación, fuente, versión de esquema, inmutabilidad, `occurredAt` tardío y rechazo de agregación no aprobada.  
**Criterio de completitud:** Sí si las pruebas demuestran catálogo cerrado, deduplicación, idempotencia, inmutabilidad y disponibilidad de los datos para KPI-01 y KPI-02 sin implementar una fórmula, semana o zona temporal no aprobada; no en caso contrario.

### T-026: Implementar puertas de calidad y seguridad para integración

**HU relacionada:** HU-015  
**Componente:** Web Client; módulos backend; Audit & Observability  
**Prioridad:** Must  
**Tamaño:** L (1-2h)  
**Dependencias:** T-009, T-019, T-021, T-022, T-025, T-030, T-031, T-032  
**Descripción:** Configurar las verificaciones de integración para exigir por separado al menos 80 % de cobertura de líneas en backend y frontend, pruebas automatizadas de la lógica modificada, SAST, evidencia de controles aplicables de OWASP ASVS 5.0.0 nivel 1, accesibilidad WCAG 2.2 AA y pruebas de las dependencias permitidas entre módulos. Bloquear hallazgos críticos o altos; para riesgos medios y controles no aplicables, exigir aprobación o justificación humana documentada. Separar explícitamente las verificaciones automatizables de las revisiones que requieren juicio humano.  
**Evidencia requerida:**

1. Reporte CI de cobertura de líneas backend igual o superior a 80 %.
2. Reporte CI de cobertura de líneas frontend igual o superior a 80 %.
3. Reporte SAST sin hallazgos críticos ni altos abiertos.
4. Matriz OWASP ASVS 5.0.0 nivel 1 con evidencia por control aplicable y justificación humana aprobada para cada control no aplicable.
5. Reporte automatizado de etiquetas y contraste, más acta de revisión humana de teclado, orden de foco y foco visible en los flujos principales.
6. Resultado de pruebas de arquitectura que rechacen dependencias prohibidas entre módulos.

Todo riesgo medio aceptado debe registrar riesgo, responsable, aprobación humana, fecha y evidencia evaluada.  
**Criterio de completitud:** Sí si CI conserva los seis artefactos, bloquea la integración al incumplir cualquier umbral o puerta automática y exige las aprobaciones o actas humanas aplicables. Una declaración documental, una evidencia incompleta o resultados ejecutados únicamente por un agente no satisfacen la tarea; no en caso contrario.

## Tareas Should

### T-027: Mostrar vencimientos y estados pendientes con claridad

**HU relacionada:** HU-003, HU-005, HU-009, HU-010  
**Componente:** Web Client  
**Prioridad:** Should  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-019  
**Descripción:** Añadir mensajes explícitos para vencimiento de oferta, preevaluación y simulación, además de la última lectura confirmada en resultados pendientes. La presentación no debe habilitar acciones vencidas ni inferir resultados desde caché.  
**Criterio de completitud:** Sí si cada entidad muestra su vencimiento/pendencia y ninguna acción prohibida se habilita por el mensaje; no en caso contrario.

### T-028: Visualizar fallas recuperables y revisión manual

**HU relacionada:** HU-012, HU-009, HU-010  
**Componente:** Web Client; Disbursement & Recovery  
**Prioridad:** Should  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-014, T-018, T-019, T-024, T-025  
**Descripción:** Incorporar vistas enriquecidas para fallo recuperable, contratación pendiente y revisión manual, con correlación y próxima acción permitida. Consumir únicamente las operaciones autorizadas por T-024 y T-025; esta tarea no crea capacidades de resolución ni permisos de negocio.  
**Criterio de completitud:** Sí si los tres estados son distinguibles y la UI solo ofrece acciones autorizadas por el estado durable; no en caso contrario.

## Tareas Could

### T-029: Añadir historial propio de simulaciones

**HU relacionada:** HU-004, HU-005  
**Componente:** Web Client; Financial Simulation  
**Prioridad:** Could  
**Tamaño:** M (30 min - 1h)  
**Dependencias:** T-010, T-019  
**Descripción:** Añadir una consulta y vista del historial de simulaciones del cliente, mostrando versión, vigencia y estado. Mantener pertenencia y no convertir selección, comparación o visualización en aceptación expresa.  
**Criterio de completitud:** Sí si el cliente solo ve sus simulaciones y abrir una entrada no crea ni modifica una aceptación; no en caso contrario.

## Resumen

- **Total:** 33 tareas.
- **Por prioridad:** 30 Must, 2 Should, 1 Could.
- **Por tamaño:** 1 S, 9 M, 23 L.
- **Por tipo:** 6 Setup/enablers (`T-000`–`T-004`, `T-030`), 17 Features (`T-005`–`T-008`, `T-010`–`T-018`, `T-023`–`T-025`, `T-029`), 3 Integración/experiencia (`T-019`, `T-027`, `T-028`), 5 Testing (`T-009`, `T-021`, `T-022`, `T-031`, `T-032`) y 2 Calidad/observabilidad (`T-020`, `T-026`).
- **Por componente** (una tarea puede contar en más de un componente): Web Client 9; Identity & Access 6; Opportunity & Application 6; Financial Simulation 7; Acceptance & Contracting 9; Disbursement & Recovery 9; Audit & Observability 12; `web-api` 2; módulos backend 3.

### Ruta crítica

`T-000 → T-030 → T-001 → T-002 → T-003 → T-004 → T-005 → T-007 → T-010 → T-011 → T-012 → T-013 → T-016 → T-017 → T-025 → T-031 → T-026`

La rama financiera `T-001 → T-008 → T-009 → T-010`, la rama del core `T-003 → T-015 → T-016`, la autorización adicional `T-007 → T-023`, los reintentos controlados `T-007/T-010/T-014/T-017 → T-024` y la evidencia KPI `T-006/T-013/T-020 → T-032` convergen en la puerta final T-026. Las pruebas Must de cierre son `T-021`, `T-022`, `T-031`, `T-032` y las pruebas incluidas en T-023–T-025; su evidencia debe ser ejecutada y adjuntada por una persona responsable o CI autorizado antes de integrar.
