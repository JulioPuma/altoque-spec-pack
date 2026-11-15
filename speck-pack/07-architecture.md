# Altoque — Arquitectura inicial

**Archivo:** `07-architecture.md`  
**Versión:** 0.1  
**Estado:** Aprobada para la primera evaluación; conserva artefactos técnicos pendientes identificados en §8  
**País de operación:** Perú  
**Fuentes normativas:** `00-constitucion.md`, `01-project-brief.md`, `02-alcance.md` y `06-user-stories.md`

> Este documento no aprueba reglas de negocio pendientes. Ante una contradicción, prevalece la Constitución. Los contratos HTTP aquí descritos son la base para el OpenAPI obligatorio; antes de implementar deben publicarse en un archivo OpenAPI versionado.

## 1. Overview del sistema

Altoque es una aplicación web responsive para que clientes sintéticos en Perú consulten oportunidades, simulen un crédito de consumo en PEN, acepten condiciones transparentes, contraten y consulten el resultado de un desembolso simulado. Se propone un monolito modular en Java/Spring Boot con una SPA Angular, una base de datos relacional transaccional y procesos internos durables para recuperación y despacho. El backend es la única autoridad para reglas financieras, estados, vigencias, pertenencia, reservas, contratos y desembolsos. El diseño prioriza integridad e idempotencia sobre respuestas optimistas: una caída o timeout nunca se interpreta por sí solo como éxito o fallo.

### 1.1 Diagrama de arquitectura

```text
┌──────────────────────────────────────────────────────────────┐
│ Android gama baja / navegador web                           │
│ Angular SPA responsive + caché solo de assets estáticos     │
└───────────────────────┬──────────────────────────────────────┘
                        │ HTTPS / JSON / gzip o Brotli
                        ▼
┌──────────────────────────────────────────────────────────────┐
│ Spring Boot — monolito modular                              │
│                                                             │
│  API/Auth ─┬─ Oportunidades y preevaluación                 │
│            ├─ Simulación y cálculo financiero              │
│            ├─ Aceptación y contratación                    │
│            ├─ Consulta de operaciones                      │
│            └─ Auditoría e instrumentación                  │
│                         │                                    │
│              Despachador/recuperadores durables             │
└───────────────┬─────────┴──────────────────┬─────────────────┘
                │ JDBC/JPA                  │ HTTP interno
                ▼                           ▼
┌──────────────────────────────┐  ┌───────────────────────────┐
│ PostgreSQL                   │  │ Core simulado             │
│ datos, estados, outbox,      │  │ ejecutar/consultar por    │
│ reservas y auditoría         │  │ operacionId idempotente   │
└──────────────────────────────┘  └───────────────────────────┘
```

No se incluye un API Gateway, broker de mensajes ni microservicios en la primera evaluación. El despachador lee una orden durable en PostgreSQL (patrón outbox/cola transaccional) y el core simulado deduplica por `operacionId`.

### 1.2 Principios de diseño

1. **Integridad antes que disponibilidad aparente.** No inferir resultados ni liberar reservas sin evidencia durable concluyente.
2. **Backend autoritativo.** Angular presenta datos y comandos; no reproduce fórmulas financieras ni transiciones de dominio.
3. **Modularidad con transacción local.** Límites internos claros, un despliegue y una base relacional permiten confirmar atómicamente contrato, auditoría, orden y consumo.
4. **Idempotencia explícita.** Los comandos con efectos aceptan una clave de idempotencia, comparan la huella de la solicitud y devuelven el recurso original ante una repetición idéntica.
5. **Estados independientes e inmutabilidad terminal.** Solicitud, simulación, contratación y desembolso poseen identidad, versión y estado propios.
6. **Concurrencia determinista.** Bloqueo de oportunidad antes de línea; revalidación completa después de esperar; control optimista por `version` en cada transición.
7. **Experiencia resiliente y liviana.** Presupuesto reducido de JavaScript, respuestas compactas, reintento solo de lecturas seguras y continuidad mediante consulta durable después de reconectar.
8. **Seguridad y privacidad por defecto.** Pertenencia validada en servidor, mínima revelación, datos sintéticos, secretos fuera del repositorio y auditoría sin contraseñas ni tokens.

## 2. Componentes principales

| Componente | Responsabilidad | Tecnología | HUs |
|---|---|---|---|
| Web Client | Autenticación, selección de oportunidad, formularios, presentación del cronograma, aceptación expresa, contratación y consulta posterior; accesible por teclado y adaptable a pantallas pequeñas. | Angular, TypeScript, Angular Router, formularios reactivos; HTML semántico y CSS responsive. | HU-001–HU-010 |
| Identity & Access | Autenticar cuentas sintéticas precargadas, emitir/invalidar sesión, aplicar roles y pertenencia sin revelar recursos ajenos. | Spring Security; contraseña con Argon2id o BCrypt; cookie de sesión `HttpOnly`, `Secure`, `SameSite=Lax`; protección CSRF. | HU-001, HU-010, HU-012, HU-014 |
| Opportunity & Application | Listar ofertas preferenciales propias; crear una única solicitud regular activa; versionar preevaluaciones y autorizaciones; controlar vigencias UTC. | Spring Boot, Java, JPA/JDBC, PostgreSQL. | HU-002, HU-003, HU-011, HU-013 |
| Financial Simulation | Validar monto/plazo, congelar TEA y seguro, calcular cuota, cronograma, TCEA y costo total con resultados deterministas. | Java `BigDecimal`, `MathContext(34, HALF_UP)`; módulo de dominio sin dependencia de Angular. | HU-004, HU-005, HU-013 |
| Acceptance & Contracting | Registrar evidencia de aceptación; crear intento y reserva; serializar oportunidad/línea; confirmar el conjunto contractual atómico; recuperar commit incierto. | Spring Boot transactions, PostgreSQL constraints/row locks y control de versión. | HU-005, HU-006, HU-007, HU-012, HU-013 |
| Disbursement & Recovery | Adquirir exclusivamente la orden, invocar el core simulado con el mismo `operacionId`, consultar y reentregar de forma idempotente; escalar revisión manual. | Scheduler/worker Spring, outbox en PostgreSQL, cliente HTTP con timeouts y backoff acotado. | HU-008, HU-009, HU-010, HU-014 |
| Audit & Observability | Registrar cada transición y evento requerido, correlacionar operaciones, producir métricas técnicas y conservar eventos KPI sin completar definiciones abiertas. | Tablas append-only, logs JSON estructurados, Micrometer/OpenTelemetry; datos sensibles enmascarados. | HU-005–HU-010, HU-012, HU-014, HU-015 |

### 2.1 Reglas de dependencia entre módulos

- `web-api` depende de casos de uso, nunca de repositorios concretos.
- `financial-domain` no depende de web, persistencia ni reloj del sistema; recibe un `Clock` y configuración versionada.
- `contracting` puede leer simulación/aceptación y línea mediante puertos internos, pero ningún otro módulo crea contratos u órdenes.
- `disbursement` solo procesa órdenes creadas por `contracting`; no cambia contratos ni libera consumo.
- `audit` participa en la transacción cuando la auditoría es obligatoria. Un log técnico no sustituye el registro auditable.
- Las integraciones de preevaluación y core simulado se aíslan detrás de adaptadores para no contaminar el dominio con DTO externos.

## 3. Modelo de datos

Todos los identificadores son UUID opacos. Los instantes se almacenan como `timestamptz` en UTC y los límites de vigencia son exclusivos. Los importes usan `numeric(19,2)` y tasas internas una precisión compatible con los cálculos aprobados; nunca `float`/`double`.

### 3.1 Entidades y atributos clave

| Entidad | Atributos clave | Restricciones y relaciones |
|---|---|---|
| `Customer` | `customerId`, `syntheticDniHash`, `passwordHash`, `status`, `createdAt` | Un DNI sintético normalizado por cuenta; no almacenar contraseña reversible. 1:1 con `CreditLine`. |
| `CreditLine` | `creditLineId`, `customerId`, `currency=PEN`, `limitAmount`, `consumedAmount`, `reservedAmount`, `version` | Límite > 0 y ≤ 50 000; saldo = límite − consumo − reservas; bloqueada después de oportunidad. |
| `PreferredOffer` | `offerId`, `customerId`, `maxAmount`, `currency`, `tea`, `issuedAt`, `expiresAt`, `status`, `version` | Identidad de emisión inmutable; máximo un contrato por `offerId`; vencimiento exclusivo a 30 días. |
| `LoanApplication` | `applicationId`, `customerId`, `status`, `createdAt`, `closedAt`, `version` | Máximo una solicitud regular activa por cliente; una identidad cerrada no se reutiliza. |
| `PreEvaluation` | `resultId`, `applicationId`, `resultVersion`, `status`, `maxAmount`, `currency`, `evaluationId`, `createdAt`, `expiresAt`, `isActive` | Versionada; 24 horas; `NO_DISPONIBLE` no reemplaza una conclusión vigente. |
| `Simulation` | `simulationId`, `opportunityType`, `offerId?`, `applicationId?`, `preEvaluationVersion?`, `customerId`, `principal`, `termMonths`, `tea`, `insurancePerInstallment`, `monthlyRate`, `financialPayment`, `totalPaymentPerInstallment`, `tcea`, `totalCost`, `status`, `createdAt`, `expiresAt`, `conditionsHash`, `version` | Exactamente una oportunidad; vigencia 30 minutos; valores financieros congelados; 1:N con cuotas. |
| `Installment` | `simulationId`, `sequence`, `dueDate`, `openingBalance`, `interest`, `principal`, `financialPayment`, `insurance`, `totalPayment`, `closingBalance` | PK (`simulationId`, `sequence`); cronograma inmutable después de simulación exitosa. |
| `Acceptance` | `acceptanceId`, `customerId`, `simulationId`, `conditionsHash`, `acceptedAt`, `expiresAt`, `evidence`, `version` | Única por (`customerId`, `simulationId`, `conditionsHash`); no se actualizan condiciones. |
| `ContractAttempt` | `attemptId`, `customerId`, `opportunityType`, `offerId?`, `applicationId?`, `acceptanceId`, `idempotencyKey`, `requestHash`, `status`, `createdAt`, `updatedAt`, `reviewAfter`, `version` | Misma clave+hash devuelve el intento; misma clave+hash distinto es conflicto; bloquea otro intento incierto. |
| `CreditReservation` | `reservationId`, `attemptId`, `creditLineId`, `amount`, `status`, `createdAt`, `convertedAt?`, `releasedAt?`, `version` | 1:1 con intento; `ACTIVE → CONSUMED|RELEASED` una sola vez. |
| `LoanContract` | `contractId`, `attemptId`, `customerId`, `offerId?`, `applicationId?`, `acceptanceId`, `conditionsSnapshot`, `signedAt`, `version` | Único por `attemptId`; índice único parcial por `offerId` o `applicationId`; snapshot exacto aceptado. |
| `DisbursementOrder` | `orderId`, `contractId`, `operationId`, `payloadHash`, `status`, `leaseOwner?`, `leaseUntil?`, `createdAt`, `updatedAt`, `version` | 1:1 con contrato; `operationId` y `contractId` únicos; adquisición exclusiva desde `DESEMBOLSO_SOLICITADO`. |
| `DisbursementEvidence` | `evidenceId`, `orderId`, `source`, `result`, `payload`, `observedAt`, `correlationId` | Append-only; permite conservar respuestas contradictorias sin cambiar un terminal. |
| `StateTransition` | `transitionId`, `aggregateType`, `aggregateId`, `operation`, `customerId`, `actorId`, `actorType`, `fromState`, `toState`, `reasonCode`, `occurredAt`, `correlationId`, `entityVersion` | Append-only; obligatorio para toda transición; índice por agregado y fecha. |
| `IdempotencyRecord` | `customerId`, `scope`, `key`, `requestHash`, `resourceType`, `resourceId`, `createdAt`, `expiresAt?` | Único por (`customerId`, `scope`, `key`); nunca guarda credenciales. |
| `InteractionEvent` | `eventId`, `eventType`, `customerId`, `opportunityType`, `offerId?`, `applicationId?`, `simulationId?`, `acceptanceId?`, `contractId?`, `interactionId?`, `occurredAt`, `correlationId`, `sourceOperation`, `sourceVersion`, `deduplicationKey`, `schemaVersion`, `payload` | Append-only; tipos limitados a RN-261; clave técnica única por evento y clave de negocio para primera visualización. Campos obligatorios por tipo, tamaño y esquema validados; no agrega fórmulas KPI pendientes. |

### 3.2 Relaciones principales

```text
Customer 1──1 CreditLine
Customer 1──N PreferredOffer
Customer 1──N LoanApplication 1──N PreEvaluation
(PreferredOffer XOR LoanApplication) 1──N Simulation 1──N Installment
Simulation 1──0..1 Acceptance
Acceptance 1──N ContractAttempt 1──1 CreditReservation
ContractAttempt 1──0..1 LoanContract 1──1 DisbursementOrder
DisbursementOrder 1──N DisbursementEvidence
Todos los agregados 1──N StateTransition
```

Una simulación conserva como máximo una aceptación para su misma versión íntegra; repeticiones idénticas devuelven esa aceptación. Puede existir más de una simulación aceptada para la misma oportunidad, pero solo un contrato.

### 3.3 Integridad transaccional y concurrencia

1. **Preparar contratación:** bloquear oportunidad; revalidar pertenencia, estados, versiones, vigencias, aceptación, unicidad e intentos inciertos; bloquear línea; revalidar saldo/version; insertar intento y reserva en una transacción.
2. **Confirmar contrato:** volver a adquirir oportunidad y luego línea; revalidar; insertar `LoanContract`, `StateTransition` obligatorio y `DisbursementOrder`, convertir la reserva a consumo y marcar el intento exitoso en una sola transacción.
3. **Despachar:** adquirir una orden elegible con bloqueo de fila y semántica equivalente a `FOR UPDATE SKIP LOCKED`; confirmar `DESEMBOLSO_PROCESANDO` antes de la primera llamada externa.
4. **Transicionar:** cada `UPDATE` incluye `WHERE id=:id AND status=:expected AND version=:version`; cero filas obliga a releer y revalidar.
5. **Terminales:** no se actualizan; una discrepancia crea evidencia, transición de escalamiento o ajuste con identidad propia.

### 3.4 Seguridad del dato

- Usar solo datos sintéticos. Separar identificadores de credenciales; el DNI sintético no es contraseña ni fuente de ella.
- Hash de contraseñas con sal y parámetros versionados. No registrar contraseñas, cookies, tokens CSRF ni cabeceras de autorización.
- Aplicar autorización por `customerId` derivado de la sesión, nunca recibido como autoridad desde el JSON.
- Aplicar mínimo privilegio a los roles `CLIENTE_SINTETICO`, `OPERADOR_REVISION_CONTRACTUAL`, `OPERADOR_REVISION_DESEMBOLSO`, `AUDITOR` y `SERVICIO_INTERNO`; la matriz técnica de permisos y los parámetros de sesión se aprueban mediante T-030 antes de publicar OpenAPI o implementar autenticación.
- Para un recurso ajeno responder `404` con el mismo cuerpo que para uno inexistente; no filtrar estados ni tiempos distintivos de forma deliberada.
- Cifrado TLS en tránsito; cifrado de volumen/backups y gestión de secretos fuera del repositorio. La política operativa de llaves debe definirse antes de un entorno con datos reales.
- Auditoría append-only con permisos de escritura restringidos a la aplicación y lectura por rol; retención y anonimización quedan sujetas a aprobación humana antes de producción.
- `conditionsSnapshot`, respuestas del core y evidencia deben validarse por esquema y limitarse en tamaño; no serializar objetos Java arbitrarios.

## 4. Contratos API

### 4.1 Convenciones

- Base: `/api/v1`; HTTPS y `Content-Type: application/json`.
- JSON usa nombres en inglés conforme a la Constitución; la documentación permanece en español.
- Fechas ISO 8601 UTC, por ejemplo `2026-11-12T18:30:00Z`; importes y tasas se transportan como cadenas decimales para evitar pérdida de precisión.
- Los comandos con efectos requieren `Idempotency-Key` (UUID) y `X-CSRF-Token`; todas las respuestas incluyen `X-Correlation-Id`.
- La sesión se mantiene en cookie segura. El cliente nunca envía `customerId` para decidir pertenencia.
- Error uniforme:

```json
{
  "code": "SIMULATION_EXPIRED",
  "message": "La simulación ya no está vigente.",
  "correlationId": "01JD...",
  "details": [{"field": "simulationId", "reason": "EXPIRED"}]
}
```

`400` JSON inválido, `401` sin sesión, `403` acción no autorizada sin referencia a recursos, `404` recurso propio inexistente o ajeno, `409` estado/idempotencia/conflicto, `422` regla de negocio, `429` límite de solicitudes y `503` dependencia no concluyente.

### 4.2 Autenticar cuenta sintética

`POST /api/v1/auth/sessions`

```json
{
  "syntheticDni": "90000001",
  "password": "example-only-not-a-real-secret"
}
```

```json
{
  "session": {"expiresAt": "2026-11-12T23:00:00Z"},
  "customer": {"displayName": "Cliente de prueba"}
}
```

Estados: `201` sesión creada y cookie emitida; `400` formato inválido; `401` credenciales inválidas con mensaje genérico; `423` cuenta sintética bloqueada; `429` límite excedido. Nunca devolver hash ni DNI completo.

### 4.3 Listar oportunidades propias

`GET /api/v1/opportunities`

```json
{
  "preferredOffers": [
    {
      "offerId": "35d5f149-00fd-4dc4-940f-c2abbbe61008",
      "maxAmount": "8000.00",
      "currency": "PEN",
      "tea": "0.180000000000",
      "expiresAt": "2026-12-01T00:00:00Z",
      "status": "VIGENTE"
    }
  ],
  "activeApplication": null
}
```

Estados: `200` incluso con listas vacías; `401`; `503` si no puede obtenerse una lectura durable. La consulta no crea automáticamente una solicitud regular.

### 4.4 Iniciar solicitud regular

`POST /api/v1/applications`  
Cabecera: `Idempotency-Key: 4bf...`

Request: `{}`

```json
{
  "applicationId": "f8854de0-d884-4634-926a-a75eb739b313",
  "status": "SOLICITUD_VALIDANDO",
  "version": 1,
  "createdAt": "2026-11-12T18:30:00Z"
}
```

Estados: `201` nueva; `200` repetición idéntica; `409 ACTIVE_APPLICATION_EXISTS` o misma clave con datos distintos; `503` preevaluación no concluyente, conservando la solicitud durable. La preevaluación puede resolverse en la respuesta o consultarse en `GET /api/v1/applications/{applicationId}`.

`POST /api/v1/applications/{applicationId}/cancellations`  
Cabeceras: `Idempotency-Key`, `X-Correlation-Id` y CSRF.

Request:

```json
{ "reasonCode": "CUSTOMER_WITHDREW" }
```

Solo el cliente propietario puede invocarlo. Adquiere el bloqueo de oportunidad y revalida estado, versión y ausencia de simulaciones, intentos, reservas y contrato. Devuelve `200` para la cancelación confirmada o su repetición equivalente; `401`; `404` con mínima revelación; `409` para clave reutilizada, versión concurrente o entidades dependientes; `422` para estado no cancelable. La respuesta conserva `applicationId`, `SOLICITUD_CANCELADA`, versión e instante auditado.

### 4.5 Crear simulación

`POST /api/v1/simulations`  
Cabecera: `Idempotency-Key: 82c...`

```json
{
  "opportunity": {
    "type": "PREFERRED_OFFER",
    "id": "35d5f149-00fd-4dc4-940f-c2abbbe61008"
  },
  "principal": "5000.00",
  "termMonths": 12
}
```

```json
{
  "simulationId": "8c37fc54-588d-4d19-8d99-a32a05697a42",
  "status": "SIMULACION_EXITOSA",
  "currency": "PEN",
  "principal": "5000.00",
  "termMonths": 12,
  "tea": "0.180000000000",
  "insurancePerInstallment": "10.00",
  "financialPayment": "451.11",
  "totalPaymentPerInstallment": "461.11",
  "tcea": "0.235678901234",
  "totalCost": "5533.32",
  "createdAt": "2026-11-12T18:30:00Z",
  "expiresAt": "2026-11-12T19:00:00Z",
  "conditionsVersion": "sha256:ab12...",
  "schedule": [
    {
      "number": 1,
      "dueDate": "2026-12-12",
      "openingBalance": "5000.00",
      "interest": "69.44",
      "principal": "381.67",
      "financialPayment": "451.11",
      "insurance": "10.00",
      "totalPayment": "461.11",
      "closingBalance": "4618.33"
    }
  ]
}
```

El ejemplo es estructural y no un vector normativo; el backend debe devolver las 12 cuotas calculadas. Estados: `201`; `200` repetición idéntica; `404` oportunidad ajena/inexistente; `409` clave reutilizada o versión reemplazada; `422` monto, plazo, saldo, dato requerido o estado inválido; `503` dependencia necesaria no disponible. Solo `SIMULACION_EXITOSA` se publica como simulación contratable.

### 4.6 Aceptar simulación íntegra

`POST /api/v1/simulations/{simulationId}/acceptances`  
Cabecera: `Idempotency-Key: e11...`

```json
{
  "conditionsVersion": "sha256:ab12...",
  "expressAction": "ACCEPT"
}
```

```json
{
  "acceptanceId": "c9dd1d86-e679-4846-884a-b2462e538b05",
  "simulationId": "8c37fc54-588d-4d19-8d99-a32a05697a42",
  "conditionsVersion": "sha256:ab12...",
  "acceptedAt": "2026-11-12T18:35:00Z",
  "expiresAt": "2026-11-12T19:00:00Z"
}
```

Estados: `201`; `200` repetición de la misma aceptación; `404`; `409` contratación ya bloqueó/cerró la oportunidad o versión no coincide; `410` no se usa para ocultar pertenencia: una simulación propia vencida devuelve `422 SIMULATION_EXPIRED`; `422` simulación no exitosa, vencida o invalidada.

### 4.7 Iniciar contratación

`POST /api/v1/contract-attempts`  
Cabecera obligatoria: `Idempotency-Key: b64...`

```json
{
  "acceptanceId": "c9dd1d86-e679-4846-884a-b2462e538b05"
}
```

```json
{
  "attemptId": "793b2e91-d2df-4515-a36c-c3485469cf02",
  "status": "CONTRATACION_PENDIENTE",
  "reservation": {"status": "ACTIVE", "amount": "5000.00", "currency": "PEN"},
  "contract": null,
  "createdAt": "2026-11-12T18:36:00Z",
  "version": 1
}
```

Si la confirmación atómica ya terminó, la misma respuesta puede incluir:

```json
{
  "attemptId": "793b2e91-d2df-4515-a36c-c3485469cf02",
  "status": "CONTRATACION_EXITOSA",
  "reservation": {"status": "CONSUMED", "amount": "5000.00", "currency": "PEN"},
  "contract": {"contractId": "b1ed78a0-841d-423b-9854-c40350cb81d9"},
  "disbursement": {
    "operationId": "64fabf82-98a9-479b-80f5-3089df4a62cb",
    "status": "DESEMBOLSO_SOLICITADO"
  },
  "version": 2
}
```

Estados: `201` intento creado; `200` repetición idéntica o intento existente; `404`; `409 IDEMPOTENCY_KEY_REUSED`, `CONTRACT_ALREADY_EXISTS` o intento incierto existente; `422` aceptación/vigencia/saldo/estado inválido; `503` resultado contractual todavía no concluyente. Ante respuesta perdida, el cliente consulta el mismo intento y no crea una clave nueva para evadir guardas.

### 4.8 Consultar intento u operación

`GET /api/v1/contract-attempts/{attemptId}`

Devuelve la representación de 4.7 con el último estado y versión durables. Estados: `200`, `401`, `404`, `503`. Es lectura pura y nunca inicia una transición.

`GET /api/v1/operations?cursor={opaque}&limit=20`

```json
{
  "items": [
    {
      "attemptId": "793b2e91-d2df-4515-a36c-c3485469cf02",
      "contractId": "b1ed78a0-841d-423b-9854-c40350cb81d9",
      "operationId": "64fabf82-98a9-479b-80f5-3089df4a62cb",
      "contractStatus": "CONTRATACION_EXITOSA",
      "disbursementStatus": "DESEMBOLSO_PENDIENTE_DE_GESTION",
      "updatedAt": "2026-11-12T18:40:00Z",
      "version": 4
    }
  ],
  "nextCursor": null
}
```

Estados: `200` con lista vacía cuando corresponda; `400` cursor/límite inválido; `401`; `503`. `limit` permitido: 1–50.

### 4.9 Contrato interno del core simulado

`POST /internal/v1/disbursements`

```json
{
  "operationId": "64fabf82-98a9-479b-80f5-3089df4a62cb",
  "contractId": "b1ed78a0-841d-423b-9854-c40350cb81d9",
  "customerId": "3a40c9c7-30d0-4df4-bc85-ff034ce03830",
  "amount": "5000.00",
  "currency": "PEN"
}
```

```json
{
  "operationId": "64fabf82-98a9-479b-80f5-3089df4a62cb",
  "result": "PROCESSING",
  "recordedAt": "2026-11-12T18:36:05Z"
}
```

`GET /internal/v1/disbursements/{operationId}` devuelve `RECEIVED`, `PROCESSING`, `SUCCESSFUL`, `REJECTED`, `FAILED_DEFINITIVE` o `NOT_FOUND`. Repetir `POST` con idéntico `operationId` y payload devuelve el resultado original (`200`); con datos distintos devuelve `409 OPERATION_PAYLOAD_CONFLICT`; una creación inicial puede devolver `202`. Timeout, `5xx`, respuesta inválida y `NOT_FOUND` mantienen el estado local pendiente. El canal interno requiere autenticación de servicio y no se expone al navegador.

### 4.10 Conducta del cliente ante conectividad intermitente

- Reintentar automáticamente `GET` con backoff y jitter; no reintentar un `POST` sin conservar exactamente su `Idempotency-Key` y cuerpo.
- Tras perder la respuesta de contratación, consultar el `attemptId` conocido. Si no llegó a recibirse, reutilizar la misma clave y cuerpo.
- Mostrar “resultado pendiente” y el instante de la última lectura confirmada; nunca un éxito inferido desde caché.
- No almacenar credenciales, aceptación ni estados operativos como autoridad en `localStorage`. La caché del service worker, si se habilita, se limita a assets versionados y página de indisponibilidad.
- Mantener bundles iniciales pequeños, carga diferida por recorrido, imágenes evitables/comprimidas y tablas de cronograma adaptadas a pantalla angosta.

### 4.11 Contratos internos de revisión manual

Los endpoints de esta sección no se exponen al rol `CLIENTE_SINTETICO`. Requieren sesión de operador, protección CSRF en comandos, autorización por rol y auditoría append-only. Un actor no autorizado recibe mínima revelación.

**Revisión contractual — rol `OPERADOR_REVISION_CONTRACTUAL`:**

- `GET /internal/v1/contract-reviews?status=CONTRATACION_EN_REVISION_MANUAL&cursor={opaque}` lista únicamente casos autorizados.
- `GET /internal/v1/contract-reviews/{attemptId}` devuelve intento, reserva y evidencia autoritativa; nunca inicia una transición.
- `POST /internal/v1/contract-reviews/{attemptId}/observations` registra una observación, motivo y referencia de evidencia.
- `POST /internal/v1/contract-reviews/{attemptId}/resolutions` resuelve únicamente con el tipo de evidencia concluyente admitido por RN-184 y la versión esperada.

**Revisión de desembolso — rol `OPERADOR_REVISION_DESEMBOLSO`:**

- `GET /internal/v1/disbursement-reviews?status=DESEMBOLSO_EN_REVISION_MANUAL&cursor={opaque}` lista casos autorizados.
- `GET /internal/v1/disbursement-reviews/{orderId}` devuelve evidencia y estado durable; nunca consulta el core por sí solo.
- `POST /internal/v1/disbursement-reviews/{orderId}/observations` registra observación, motivo y correlación.
- `POST /internal/v1/disbursement-reviews/{orderId}/queries` solicita otra consulta del mismo `operationId`; no admite otro identificador ni payload.

Todos los comandos exigen `Idempotency-Key`, `X-Correlation-Id`, motivo no vacío y versión esperada. Responden `403` ante rol insuficiente, `404` con mínima revelación, `409` ante versión/conflicto idempotente y `422` cuando la acción no está permitida por el estado o la evidencia. Las respuestas y esquemas completos se publican en OpenAPI mediante T-004.

### 4.12 Captura de interacciones KPI

`POST /api/v1/interaction-events` acepta únicamente `OPPORTUNITY_VIEWED`, `SIMULATION_PARAMETERS_CHANGED` y `SCHEDULE_VIEWED` desde una sesión de cliente válida. El cliente envía `interactionId`, tipo, recurso propio y versión de esquema; el backend deriva `customerId`, valida pertenencia, asigna o valida `occurredAt` UTC y construye la clave de deduplicación. Una repetición equivalente devuelve `200`; datos distintos con la misma identidad devuelven `409`; campos o esquema inválidos devuelven `422`.

`ACCEPTANCE_CONFIRMED` y `CONTRACT_CONFIRMED` se emiten exclusivamente en backend desde la confirmación durable correspondiente y no disponen de endpoint público de creación. La persistencia del evento no autoriza calcular KPI mientras PA-05 o PA-06 sigan abiertas. Los eventos tardíos conservan su instante y nunca se reasignan silenciosamente.

## 5. ADRs

### ADR-001 — Monolito modular con PostgreSQL

**Estado:** Aceptado para la primera evaluación por aprobación humana del 2026-11-14.  
**Contexto:** La confirmación contractual exige una única transacción sobre contrato, auditoría, orden y consumo; el equipo debe entregar una primera evaluación cerrada sin necesidad aprobada de escalado independiente.  
**Decisión:** Implementar un monolito modular Spring Boot desplegable como una unidad, con PostgreSQL como fuente durable y límites de módulo comprobables.  
**Alternativas:** Microservicios con saga; funciones serverless; varias bases por dominio.  
**Consecuencias:** Simplifica atomicidad, operación y pruebas de concurrencia. Aumenta el acoplamiento de despliegue y exige disciplina para preservar límites. Si el sistema se divide después, la orden/outbox y los contratos de módulo ofrecen puntos de extracción.

### ADR-002 — Cola transaccional en base de datos para desembolso

**Estado:** Aceptado para la primera evaluación por aprobación humana del 2026-11-14.  
**Contexto:** La orden debe confirmarse junto con el contrato y solo una orden durable adquirida exclusivamente puede iniciar el despacho. Un broker separado introduciría una ventana dual-write.  
**Decisión:** Persistir `DisbursementOrder` en la misma transacción contractual y procesarla con workers que usan adquisición exclusiva, lease recuperable y control de versión.  
**Alternativas:** Publicación directa después del commit; Kafka/RabbitMQ con dual-write; XA entre base y broker.  
**Consecuencias:** El contrato nunca existe sin orden y no se necesita infraestructura adicional. Hay polling y carga controlada sobre PostgreSQL; deben medirse índices, tamaño de lote, lease y lag. Desde `DESEMBOLSO_PROCESANDO` la recuperación consulta la operación original.

### ADR-003 — Sesión servidor con cookie segura

**Estado:** Aceptado para la primera evaluación por aprobación humana del 2026-11-14; los parámetros concretos se completan en la especificación de seguridad.  
**Contexto:** Es una web cerrada con cuentas sintéticas, navegación en Android de gama baja y necesidad de revocación sencilla. Guardar bearer tokens accesibles a JavaScript aumenta el impacto de XSS.  
**Decisión:** Usar sesión opaca de servidor en cookie `HttpOnly`, `Secure`, `SameSite=Lax`, rotación al autenticar y protección CSRF para comandos.  
**Alternativas:** JWT en `localStorage`; JWT en cookie; OAuth/OIDC externo.  
**Consecuencias:** Reduce exposición del token y simplifica revocación; requiere almacenamiento/afinidad compartida si hay varias instancias y control CSRF. Un proveedor OIDC podrá adoptarse mediante ADR si el alcance productivo lo requiere.

### ADR-004 — Cálculo financiero determinista en un módulo Java

**Estado:** Aceptado por derivación directa de la Constitución; implementación sujeta a pruebas con vectores normativos.  
**Contexto:** Las condiciones mostradas, aceptadas y contratadas deben ser idénticas; JavaScript usa números binarios y no debe mantener fórmulas paralelas.  
**Decisión:** Implementar todas las fórmulas en `financial-domain` con `BigDecimal`, precisión 34 y `HALF_UP`, congelando entradas y persistiendo el cronograma y hash de condiciones. Angular solo renderiza la respuesta.  
**Alternativas:** Cálculo en frontend; biblioteca financiera externa como autoridad; cálculo bajo demanda sin snapshot.  
**Consecuencias:** Resultado reproducible y auditable; mayor costo de pruebas numéricas y de migración si cambia una regla. Cada cambio requiere nueva versión de simulación, no mutación.

### ADR-005 — Idempotencia persistente y bloqueo oportunidad–línea

**Estado:** Aceptado por derivación de los invariantes INV-03, INV-11, INV-12, INV-15 e INV-16.  
**Contexto:** Conectividad intermitente, doble toque y concurrencia entre oportunidades pueden duplicar aceptaciones, intentos o consumo.  
**Decisión:** Persistir claves de idempotencia con hash canónico; usar restricciones únicas; bloquear siempre oportunidad y luego línea; revalidar todas las guardas tras esperar; transicionar con estado esperado y versión.  
**Alternativas:** Idempotencia solo en memoria; bloqueo distribuido; aislamiento serializable global.  
**Consecuencias:** Los reintentos son seguros y se evita deadlock por orden inverso. Aumenta la complejidad de índices, respuestas de conflicto y pruebas concurrentes. Una clave nueva no evade invariantes de negocio.

### ADR-006 — Recuperación por evidencia durable, sin circuitos de éxito/fallo inferido

**Estado:** Aceptado por derivación de INV-08, INV-13, INV-14 e INV-17.  
**Contexto:** Una respuesta puede perderse después de confirmar una transacción o de ejecutar el core simulado.  
**Decisión:** Consultar el intento/operación original; reentregar solo el comando original con el mismo identificador y payload cuando la política lo permita; conservar pendiente ante ambigüedad y escalar a revisión manual.  
**Alternativas:** Marcar fallo al timeout; crear otra operación; compensar automáticamente el contrato.  
**Consecuencias:** Evita duplicidad y falsos negativos, pero algunos casos permanecen pendientes y requieren observabilidad y operación manual. Para contratación: consulta cada 30 segundos hasta 15 minutos y luego revisión manual con objetivo de cuatro horas; el tiempo nunca acredita un resultado.

## 6. Restricciones técnicas y de calidad

### 6.1 Stack y versiones

- Backend obligatorio: **Java + Spring Boot**.
- Frontend obligatorio: **Angular + TypeScript**.
- Persistencia aprobada para la primera evaluación: PostgreSQL relacional con migraciones versionadas.
- Código e identificadores en inglés; documentación en español; estados de dominio conservan su catálogo canónico.
- Las versiones exactas y compatibles de JDK, Spring Boot, Angular, TypeScript, PostgreSQL, drivers y herramientas **deben fijarse en el manifiesto de construcción y aprobarse antes de implementar**. Este documento no inventa versiones aún no aprobadas.
- Sustituir Java, Spring Boot o Angular requiere ADR y aprobación humana.

### 6.2 Restricciones funcionales implementadas por arquitectura

- Moneda única PEN; monto S/ 500–S/ 10 000 en incrementos de S/ 100; plazos 6, 12, 18 o 24, además de máximo y saldo.
- TEA de oferta o tasa pizarra configurada; seguro fijo versionado; ausencia de un dato requerido bloquea la operación.
- Simulación por sistema francés y TCEA conforme a `02-alcance.md` §5, exclusivamente en backend.
- Oferta preferencial: 30 días; preevaluación/oferta base: 24 horas; simulación: 30 minutos; límites exclusivos en UTC.
- Cualquier cliente sintético puede iniciar explícitamente el flujo regular aunque tenga una oferta preferencial vigente. Ambos recorridos son oportunidades independientes, no se invalidan entre sí por ese inicio y comparten la misma línea; el flujo regular nunca se inicia automáticamente.
- Una solicitud solo se cancela por su cliente propietario desde `SOLICITUD_INICIADA` o `SOLICITUD_HABILITADA` y antes de cualquier simulación. La operación se serializa por oportunidad, es idempotente y auditable; cualquier entidad dependiente o estado distinto bloquea la cancelación.
- Una aceptación por cliente, simulación y versión inmutable; un contrato por `offerId` o `applicationId`.
- Cuatro máquinas separadas y terminales inmutables.
- Intento y reserva durables antes de registrar; contrato, auditoría, orden y conversión en una transacción.
- Una orden y un `operationId` por contrato; timeout o `NOT_FOUND` no son terminales.
- Solo cuentas y datos sintéticos; sin registro autónomo, RENIEC, scoring real, dinero real ni productos fuera de consumo.

### 6.3 Calidad, seguridad y accesibilidad

- Cobertura de líneas mínima del 80 % por separado en backend y frontend; toda lógica nueva con pruebas automatizadas.
- Pruebas obligatorias: unitarias de vectores financieros; integración transaccional; restricciones únicas; concurrencia de aceptación/contratación/reserva; idempotencia; timeout, respuesta perdida y contradicción terminal; autorización horizontal.
- SAST antes de integrar. No quedan hallazgos críticos o altos; un riesgo medio exige aceptación humana documentada.
- Seguridad verificada contra OWASP ASVS 5.0.0 nivel 1. Controles no aplicables requieren justificación aprobada.
- Flujos principales operables por teclado, etiquetas programáticas, foco visible y contraste WCAG 2.2 AA.
- Etiquetas y contraste se verifican automáticamente cuando la herramienta lo permita; teclado, orden de foco y foco visible requieren además acta de revisión humana de los flujos principales.
- Pruebas de arquitectura deben fallar ante dependencias de módulo prohibidas por §2.1.
- La puerta de integración conserva seis artefactos: cobertura backend, cobertura frontend, SAST, matriz ASVS, evidencia de accesibilidad automática/humana y pruebas de arquitectura.
- Contratos HTTP publicados en OpenAPI y validados en CI; migraciones compatibles con rollback operativo o estrategia de avance seguro.
- Logs estructurados con `correlationId`, sin secretos ni datos financieros personales reales; métricas de errores, latencia, conflictos, lag de outbox, órdenes pendientes y casos en revisión.
- Alertas para intento contractual pendiente cercano a 15 minutos, revisión superior al objetivo de cuatro horas, contradicción terminal y crecimiento anómalo de órdenes pendientes.

### 6.4 Restricción operativa para agentes de IA

Los agentes pueden leer y editar archivos autorizados, pero no ejecutar código, pruebas, scripts, aplicaciones, despliegues ni publicaciones. Una persona responsable ejecuta las verificaciones o autoriza su ejecución en CI y adjunta resultados como evidencia. Por ello, los resultados de pruebas no pueden declararse verificados únicamente a partir de este documento.

## 7. Trazabilidad resumida

La asignación normativa canónica RN → HU se mantiene en `06-user-stories.md`, sección «Matriz canónica de trazabilidad normativa». Esta sección arquitectónica resume únicamente HU → componente.

| HU | Componentes | Entidades/API principales |
|---|---|---|
| HU-001 | Web Client, Identity & Access | `Customer`; `POST /auth/sessions` |
| HU-002 | Web Client, Opportunity & Application | `PreferredOffer`; `GET /opportunities` |
| HU-003 | Opportunity & Application | `LoanApplication`, `PreEvaluation`; `POST /applications` |
| HU-004 | Financial Simulation | `Simulation`, `Installment`; `POST /simulations` |
| HU-005 | Web Client, Financial Simulation, Acceptance & Contracting | `Acceptance`; `POST /simulations/{id}/acceptances` |
| HU-006 | Acceptance & Contracting | `ContractAttempt`, `CreditReservation`; `POST /contract-attempts` |
| HU-007 | Acceptance & Contracting, Audit | `LoanContract`, `StateTransition`, `DisbursementOrder` |
| HU-008 | Disbursement & Recovery | `DisbursementOrder`, `DisbursementEvidence`; core interno |
| HU-009 | Disbursement & Recovery, Audit | evidencia, transiciones y consulta por `operationId` |
| HU-010 | Web Client, consulta, Audit | `GET /operations`, `GET /contract-attempts/{id}`, `InteractionEvent`, `POST /interaction-events` |
| HU-011 | Opportunity & Application, Audit | autorización versionada de preevaluación; `POST /applications` y consulta asociada |
| HU-012 | Acceptance & Contracting, Identity & Access, Audit | `ContractAttempt`, reserva, evidencia durable; `/internal/v1/contract-reviews` |
| HU-013 | Módulos de dominio, Audit | reintentos controlados con misma identidad y transición versionada |
| HU-014 | Identity & Access, Disbursement & Recovery, Audit | evidencia; `/internal/v1/disbursement-reviews` |
| HU-015 | CI, Web Client, módulos backend, Audit | cobertura, SAST, ASVS, accesibilidad y pruebas de arquitectura |

## 8. Decisiones y artefactos pendientes antes de implementar

1. Fijar versiones exactas compatibles del stack y documentarlas en archivos de construcción mediante T-001.
2. Elaborar y aprobar mediante T-030 la especificación de seguridad: parámetros de contraseña/sesión, CSRF, rate limits, secretos, cifrado, roles y retención.
3. Publicar mediante T-004 el OpenAPI completo, incluidos catálogos canónicos de estado, errores y endpoints internos de revisión.
4. Implementar el catálogo técnico aprobado en RN-261 sin calcular las fórmulas abiertas de PA-05 y PA-06; T-032 verifica la evidencia disponible y bloquea agregaciones no aprobadas.
5. Definir mediante T-014, T-017 y T-025 los runbooks de recuperación contractual, desembolso, contradicción terminal y revisión manual.
6. Convertir mediante T-031 los diagramas de estado y restricciones únicas en pruebas ejecutadas por una persona responsable o CI autorizado.
