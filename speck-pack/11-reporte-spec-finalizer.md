# Altoque — Reporte de auditoría del Spec Finalizer

**Archivo:** `11-reporte-spec-finalizer.md`
**Fecha:** 2026-11-14
**Alcance auditado:** `00-constitucion.md` a `10-validation-log.md` (11 archivos) y `prototipos/README.md` como referencia de P-001–P-013.
**Método:** lectura cruzada completa y verificación automatizada de IDs (cobertura RN→HU, rangos de HU/P/T/PA) y recálculo de los ejemplos financieros con `BigDecimal` 34 dígitos `HALF_UP`.

---

## Resumen ejecutivo

**Veredicto: NO LISTO para entregarse a un agente de IA implementador.** Puede quedar listo tras corregir **9 bloqueantes**, sin necesidad de rediseño.

El Spec Pack es excepcionalmente riguroso en lo financiero, la concurrencia, la idempotencia y la recuperación: las 261 RN existen y cada una tiene exactamente una HU primaria, no hay IDs huérfanos y los vectores financieros normativos son correctos. Los problemas están en otros puntos: **huecos en las máquinas de estados, contratos de API que contradicen las reglas, políticas operativas referidas pero nunca definidas y decisiones abiertas clasificadas por debajo de su impacto real.** Un agente se toparía con todos ellos en la ruta crítica (T-001, T-004, T-007, T-013, T-017 y T-024).

| Criterio | Score | Motivo principal |
|---|:-:|---|
| Operatividad | **3 / 5** | Faltan transiciones, endpoints, políticas de reintento y simuladores de preevaluación; hay contradicciones API↔RN. |
| Utilidad | **4 / 5** | Las reglas son precisas, testeables y trazables. Los vectores numéricos y los criterios Given/When/Then son de alta calidad. |
| Orden | **3 / 5** | Versiones inconsistentes (0.1/0.2 frente a v1.0), `10-validation-log.md` vacío, colisiones de IDs y mezcla de nombres en español e inglés sin tabla de correspondencia. |
| **Global** | **3,3 / 5** | |

### Bloqueantes (deben resolverse antes de implementar)

| # | Bloqueante | Dónde corregir |
|---|---|---|
| B-01 | La máquina de Solicitud no tiene transición para el **vencimiento de preevaluación** (24 h) desde `SOLICITUD_HABILITADA` ni desde `SOLICITUD_ESPERANDO_AUTORIZACION`, ni para el **cierre con contrato** (RN-175). Como RN-173 impide crear otra solicitud mientras haya una activa, el cliente queda **bloqueado indefinidamente**. | 03 §7 Solicitud, 04 estados canónicos, 05 (nueva PA) |
| B-02 | `POST /contract-attempts` (07 §4.7) devuelve `CONTRATACION_PENDIENTE` en un intento recién creado. En las RN, ese estado significa *resultado incierto* y dispara la recuperación de 15 minutos y la revisión manual (RN-181–182). Además, no se define si la contratación es síncrona o asíncrona ni qué proceso la mueve de `INICIADA` a `VALIDANDO` y a `REGISTRANDO`. | 07 §4.7, §3.3; nuevo ADR |
| B-03 | No está definida la **política de recuperación de desembolso** (intervalos, número de reentregas, timeouts HTTP, duración del lease, cuándo escalar a `DESEMBOLSO_EN_REVISION_MANUAL`). RN-134, RN-055 y 03 §7 la citan como «política técnica» inexistente. Tampoco están fijados los **límites de reintentos controlados** de RN-227 («dentro de los límites aprobados»). | 05 (nueva PA) → 07 §6 o anexo de política |
| B-04 | No existe especificación de la **fuente de preevaluación ni de la autorización adicional** en el entorno cerrado: no hay simulador, contrato interno equivalente a 07 §4.9, actor que pase `PENDIENTE` a `AUTORIZADA` ni datos sintéticos de prueba. T-007 y T-023 no se pueden implementar. | 07 §2, §4 (nuevo §4.13); 08 nueva tarea |
| B-05 | PA-33 (versiones del stack) está como **MEDIA**, pero bloquea T-001 y con ello toda la ruta crítica. PA-34 (catálogo de motivos y mensajes) bloquea el catálogo de errores de T-004 y los textos de P-007 y P-009. La tabla «Pendientes críticos y altos» aparece vacía. | 05 |
| B-06 | Faltan endpoints que la UI y las RN necesitan: `GET /simulations/{id}`, `GET /applications/{id}` (solo se menciona, sin esquema), la reconsulta de preevaluación (RN-110, RN-174), el reintento de solicitud, simulación e intento (RN-221, RN-222, RN-180), el cierre de sesión (RN-241) y la consulta de autorización adicional (HU-011). | 07 §4 |
| B-07 | El modelo de datos no representa **operadores, auditores ni identidades de servicio** (solo `Customer`). Tampoco incluye la configuración financiera versionada (RN-201, RN-207), las autorizaciones adicionales, la evidencia de presentación previa a aceptar (RN-253), las observaciones de revisión, el incidente crítico (RN-183), el expediente de conflicto (RN-236), los intentos fallidos de login ni el bloqueo temporal (RN-242). | 07 §3.1; T-002/T-003 |
| B-08 | Hay **contradicciones normativas en el glosario** que inducen a implementar mal: «Operación original… en `DESEMBOLSO_PROCESANDO` solo puede consultarse, **no reenviarse**» contradice INV-14 y RN-133. «Flujo regular: recorrido para clientes **sin** oferta preferencial» contradice RN-172. | 04 |
| B-09 | `10-validation-log.md` está **vacío** (0 líneas) y los documentos declaran versiones 0.1 o 0.2. La Constitución y 05 definen el Spec Pack oficial como **6 archivos**, no 11. Así no se puede afirmar que exista un «Spec Pack v1.0». | 00 intro, 05 intro, encabezados de 01–10 |

---

## 1. Trazabilidad completa

### 1.1 RN → HU — ✅ Completa

- Las 261 RN (RN-001–RN-261) están definidas en `03`, sin huecos.
- En la matriz canónica de `06`, **todas las RN tienen exactamente una HU primaria**: no hay faltantes ni duplicados (verificado por script).
- Observación: RN-028 y RN-080 (teclado y etiquetas) tienen como primaria a HU-015, que es la historia de CI. La responsabilidad funcional es de la UI. **Corrección:** asignar HU-001–HU-010 como primarias o crear una HU de accesibilidad del recorrido, y dejar HU-015 como secundaria (verificación).

### 1.2 HU → Pantalla — ⚠️ Casi completa

| HU | Pantallas | Estado |
|---|---|---|
| HU-001–HU-014 | Cubiertas en 09 §1 | ✅ |
| HU-015 | Ninguna (justificado: puerta de CI) | ✅ aceptable |
| HU-012 y HU-014 (operadores) | P-010–P-013 | ⚠️ **No hay pantalla de inicio de sesión interna.** P-001 es solo para clientes y 09 §5 habla de una «sesión y autorización separadas» sin especificarla. |

**Corrección:** añadir P-017 «Inicio de sesión interno», o declarar en P-001 que también la usan los roles internos, con su redirección por rol.

### 1.3 Pantalla → Tareas — ❌ Incompleta

- **El backlog no referencia ningún P-XXX** (0 ocurrencias). La relación pantalla→tarea solo se puede inferir.
- **P-010–P-013 no tienen tarea de frontend.** T-019 cubre HU-001–HU-010. T-014 y T-025 son backend y T-028 (Should) es una vista de cliente.
- P-015 (comparar) y P-016 (ayuda) no tienen tarea. P-014 sí la tiene (T-029).
- **Corrección:** añadir el campo «Pantallas» a cada tarea, crear T-033 «UI interna de revisión contractual (P-010/P-011)» y T-034 «UI interna de revisión de desembolso (P-012/P-013)», y tareas Could para P-015 y P-016 (o retirar esas pantallas del inventario).

### 1.4 Tareas → RN

Solo T-000, T-001, T-030 y T-032 tienen el campo «Requisitos relacionados». El resto depende de la matriz de `06`, que sí lista tareas por RN. Es aceptable porque la matriz es canónica, pero conviene que cada tarea liste sus RN para que un agente trabaje con una sola tarea en contexto.

---

## 2. Consistencia de IDs

| Verificación | Resultado |
|---|---|
| HU referenciadas fuera de HU-001–HU-015 | Ninguna ✅ |
| P referenciadas fuera de P-001–P-016 | Ninguna ✅ |
| T referenciadas inexistentes (T-000–T-032) | Ninguna ✅ |
| PA referenciadas no definidas en 05 | Ninguna ✅ |
| Conteos del resumen de 08 (33 tareas; 30/2/1; 1 S/9 M/23 L) | Correctos ✅ |

**Problemas:**

1. **Colisión de IDs.** `05` cita «M-04, M-08, M-09 **de QA**» y los hallazgos «C-01, C-02, C-03». Esos IDs ya existen en `02` como capacidades MUST (M-xx) y COULD (C-xx). **Corrección:** renombrar los hallazgos QA como `QA-M-xx` y `QA-C-xx`.
2. **Identificadores de dominio en dos idiomas sin correspondencia:** `ofertaId`/`offerId`, `solicitudId`/`applicationId`, `simulacionId`/`simulationId`, `intentoId`/`attemptId`, `operacionId`/`operationId`, `resultadoId`/`resultId`. 03, 06 y 09 usan español; 07 y 08 usan inglés. **Corrección:** añadir al glosario una tabla «Término de dominio → campo API/persistencia».
3. **Valores del core simulado en dos idiomas:** las RN usan `EXITOSA`, `RECHAZADA`, `FALLIDA_DEFINITIVA`, `NO_ENCONTRADA`, `RECIBIDA` y `PROCESANDO`; 07 §4.9 usa `SUCCESSFUL`, `REJECTED`, `FAILED_DEFINITIVE`, `NOT_FOUND`, `RECEIVED` y `PROCESSING`. PRINCIPIO-001 exige conservar el catálogo canónico en los valores publicados. **Corrección:** elegir uno y documentar el mapeo.
4. **Valores sin catálogo canónico:** `status: "VIGENTE"` de la oferta (07 §4.3), `ACTIVE`/`CONSUMED`/`RELEASED` de la reserva y el estado «vencida» de la solicitud (09 P-003). No existe máquina de estados de la oferta preferencial (vigente, vencida, reemplazada, contratada). **Corrección:** definir los catálogos en 04 y su origen en 03.
5. **`PENDIENTE` ambiguo:** el glosario lo define como «abreviatura descriptiva», pero también es el valor canónico de la autorización adicional (RN-158). **Corrección:** eliminar la abreviatura o renombrar el valor de autorización como `AUTORIZACION_PENDIENTE`.
6. **Las referencias DEC-xx no se pueden verificar.** 05 admite que las DEC completas no se incluyeron. Es aceptable como historial, pero no deben usarse como fuente normativa.

---

## 3. Cobertura de la Constitución

| Principio | Estado | Hallazgo y corrección |
|---|---|---|
| P-001 Idiomas | ⚠️ | Mezcla de valores del core en inglés y español (ver §2.3). |
| P-002 Stack | ❌ bloqueante | Las versiones no están fijadas (PA-33, 07 §6.1). El principio exige fijarlas «antes de implementar». |
| P-003 Cálculo centralizado | ✅ | Bien reflejado en 07 (ADR-004), 09 §0 y T-008. |
| P-004 Transparencia | ✅ | P-005 muestra los nueve elementos. Sugerencia: mostrar también la TEA, porque «Condiciones aceptadas» la incluye (04). |
| P-005 Correspondencia | ⚠️ | El hash de `conditionsVersion` no tiene canonicalización definida (RN-258 exige ADR u OpenAPI). |
| P-006 Atomicidad | ✅ / ⚠️ | 07 §3.3 es correcto. Pero RN-085 exige el «reloj UTC autoritativo de la base de datos justo antes de escribir»: en PostgreSQL `now()` devuelve el **inicio de la transacción**. **Corrección:** fijar en un ADR el uso de `clock_timestamp()` o equivalente. |
| P-007 Concurrencia | ✅ / ⚠️ | ADR-005 fija el orden, pero no el mecanismo físico (qué filas se bloquean en cada tipo de oportunidad). 03 §9 lo remite a un ADR. |
| P-008 Desembolso único | ⚠️ | Contradicción en el glosario (B-08) y política de reentrega sin definir (B-03). |
| P-009 Máquinas de estados | ❌ | Huecos en Solicitud (B-01). Además, `DESEMBOLSO_FALLIDO` aparece fuera de la lista de terminales en 03 §8 («exige RN-053»): hay que declararlo terminal explícitamente. `SIMULACION_EXPIRADA` no indica quién persiste la transición ni cuándo (evaluación perezosa o scheduler). |
| P-010 Auditoría | ⚠️ | PA-32 abierta (auditoría de creación, actor de procesos automáticos). Afecta al esquema de T-003 y T-020. |
| P-011 Calidad | ✅ | T-026 con sus seis artefactos. |
| P-012 Accesibilidad y seguridad | ✅ | Cubierto (09 §0, §4.4; T-026; T-030). |
| P-013 Datos sintéticos | ❌ | **Ninguna tarea crea los datos sintéticos** (cuentas, credenciales generadas por separado, líneas, ofertas). Es una exigencia explícita del principio y de RN-124 y RN-199. |
| P-014 Gobernanza | ⚠️ | La Constitución lista 6 archivos oficiales y hay 11. Varias decisiones ausentes no se trasladaron a 05 (B-01, B-03, B-04), en contra de la regla 3 del Gobierno de cambios. |

---

## 4. Glosario

### 4.1 Contradicciones (corregir)

| Término | Problema | Corrección |
|---|---|---|
| Operación original | «En `DESEMBOLSO_PROCESANDO` solo puede consultarse, no reenviarse» contradice INV-14 y RN-133. | «…solo puede consultarse o reentregarse idempotentemente con idéntico `operacionId` y datos». |
| Flujo regular | «Para clientes sin oferta preferencial» contradice RN-172 y 02. | «Recorrido que cualquier cliente sintético inicia explícitamente…». |
| Contratación | Menciona solo «contrato y auditoría». | Referenciar el *conjunto contractual* (4 componentes). |
| Aceptación vigente | Omite la preevaluación y la autorización (RN-113). | Alinear con RN-113. |

### 4.2 Términos usados sin definir

- **Dominio:** cronograma, consumo de línea, orden de desembolso, `operacionId`, intento incierto, versión íntegra o inmutable, incidente crítico, punto de decisión de vigencia y reloj autoritativo, versión de entidad frente a versión financiera (PA-32), reconsulta, mínima revelación, DNI sintético, TIR, semana de contratación.
- **Técnicos:** outbox, lease, `Idempotency-Key`/huella (`requestHash`), `correlationId`, SAST, OWASP ASVS, WCAG, ADR, OpenAPI, BMAD/SDD.
- **UI:** bandeja, caso, última lectura confirmada.

**Corrección:** añadir estas entradas y la tabla de correspondencia de identificadores (§2.2).

---

## 5. Preguntas abiertas

### 5.1 Registradas en 05

| PA | Prioridad actual | ¿Bloquea? | Prioridad recomendada |
|---|---|---|---|
| PA-33 Versiones del stack y criterios | MEDIA | **Sí: T-001 y toda la ruta crítica** | **CRÍTICA** |
| PA-34 Detalles visibles y catálogo de mensajes | MEDIA | **Sí: errores de T-004 y textos de P-007 y P-009** | **ALTA** |
| PA-32 Auditoría (creación, actor automático) | MEDIA | Parcialmente: esquema de T-003 y T-020 | ALTA |
| PA-31 Ajustes sobre consumo | MEDIA | No, si se declara fuera de la primera evaluación | MEDIA; declarar RN-104 no implementable en v1 |
| PA-05, PA-06 Fórmulas KPI | MEDIA | No (solo se instrumenta, RN-261) | MEDIA |

### 5.2 Decisiones ausentes no registradas (crear PA)

1. Transiciones de Solicitud por vencimiento y por cierre con contrato (B-01).
2. Contratación síncrona o asíncrona, y estado devuelto al crear el intento (B-02).
3. Política de recuperación y reentrega del desembolso, límites de reintento controlado, timeouts y lease (B-03).
4. Fuente de preevaluación y autorización en el entorno cerrado (B-04).
5. Configuración financiera: quién la cambia, cómo se versiona y si existe interfaz (RN-207 presupone cambios).
6. Persistencia de expiraciones (`SIMULACION_EXPIRADA`, oferta vencida): perezosa o por scheduler, y su auditoría.
7. Momento en que la UI registra la **evidencia de presentación** (RN-253) y cuándo se emite `SIMULATION_PARAMETERS_CHANGED`.
8. Umbral y duración del bloqueo temporal (RN-242); ya asignado a T-030, aceptable.

---

## 6. Calidad del documento de arquitectura

### 6.1 Secciones

| Sección | Estado |
|---|---|
| 1 Overview y diagrama | ✅ Completa. El core simulado aparece en el diagrama, pero no en la tabla de componentes de §2 ni en los módulos de T-001. |
| 2 Componentes | ⚠️ Faltan «Core simulado» y «Preevaluación simulada» como componentes con responsabilidad y límites. |
| 3 Modelo de datos | ❌ Faltan entidades (B-07). |
| 4 Contratos API | ❌ Hay contradicciones (B-02, y §4.4 devuelve `503` donde RN-215 exige el estado `SOLICITUD_PENDIENTE`) y faltan endpoints (B-06). |
| 5 ADRs | ⚠️ Son coherentes, pero faltan algunos (ver 6.2). |
| 6 Restricciones | ⚠️ Versiones sin fijar (PA-33). |

### 6.2 ADRs

- **Coherentes entre sí y con la Constitución:** ADR-001 (monolito + PostgreSQL) habilita la atomicidad de INV-12. ADR-002 elimina la ventana de doble escritura. ADR-003 (cookie `SameSite=Lax` + CSRF) es consistente con 07 §4.1.
- **Matiz:** ADR-004, ADR-005 y ADR-006 son «aceptados por derivación», sin aprobación humana ni fecha. Conviene registrar su aprobación igual que ADR-001–003 (T-000).
- **ADRs faltantes que las RN exigen explícitamente:**
  - ADR-007: canonicalización del hash de condiciones y del hash de la petición idempotente (RN-252, RN-258).
  - ADR-008: mecanismo físico de bloqueo (filas o advisory locks por tipo de oportunidad y línea; RN-136–143, 03 §9).
  - ADR-009: reloj autoritativo en BD (`clock_timestamp()` frente a `now()`; RN-085, RN-096).
  - ADR-010: ejecución de los recuperadores y despachadores en varias instancias (scheduler, lease, `SKIP LOCKED`).
  - ADR-011: forma de despliegue del core y de la preevaluación simulados (módulo interno o proceso separado).

### 6.3 Error en el ejemplo de §4.5

Aunque se declara «estructural», el ejemplo es **numéricamente incorrecto** y un agente podría copiarlo como fixture. Con P = 5000, TEA = 0,18, n = 12 y seguro = 10, las reglas aprobadas dan:

| Campo | Ejemplo 07 | Valor correcto |
|---|---|---|
| `financialPayment` | 451.11 | **455.23** |
| `totalPaymentPerInstallment` | 461.11 | **465.23** |
| `totalCost` | 5533.32 | **5582.79** |
| TCEA | 0.2356… | **≈ 0.229742** (22,97 %) |

Los datos de los prototipos (TEA 0,20 → cuota 459,28, TCEA 25,03 %, costo total 5 631,44) **sí son correctos**. **Corrección:** reemplazar el ejemplo por valores correctos o por marcadores `"<calculado>"`.

---

## 7. Calidad del backlog

### 7.1 Lo que está bien

Todas las tareas tienen tamaño, prioridad, dependencias, descripción y un criterio de completitud binario. La ruta crítica es explícita y los conteos del resumen cuadran.

### 7.2 Atomicidad: ❌ varias tareas no caben en «< 2 h»

| Tarea | Problema | División sugerida |
|---|---|---|
| T-019 | Todo el frontend del cliente (9 pantallas) como L de 1–2 h. | Una tarea por pantalla o por flujo (F-01–F-05), más una tarea de shell, router e interceptores. |
| T-007 | Creación de solicitud, adaptador, 4 resultados, cancelación y concurrencia. | (a) creación e idempotencia; (b) adaptador y resultados; (c) cancelación. |
| T-003 | 11 entidades y sus restricciones. | Por agregado: simulación; contratación; desembolso; auditoría y KPI. |
| T-004 | OpenAPI completo, interno y público. | Público de cliente / interno de revisión / core y preevaluación. |
| T-014 | Recuperador, rol, revisión manual y runbook. | Recuperador / endpoints de revisión / runbook. |
| T-031, T-026 | Pruebas exhaustivas de 4 máquinas / 6 puertas de CI. | Por máquina / por puerta. |

### 7.3 Dependencias

- **Inversión:** T-020 (registro de transiciones append-only) depende de T-013 y T-017, pero RN-081 exige auditar *cada* transición desde T-005 y T-007. **Corrección:** separar T-020a «infraestructura de `StateTransition`» (depende de T-003 y precede a T-005, T-007, T-010…) de T-020b «métricas y eventos KPI».
- 07 §8.5 asigna runbooks a T-017 y T-025, pero esas tareas no los incluyen como entregable.

### 7.4 Tareas faltantes

1. Datos sintéticos de prueba (cuentas, credenciales independientes del DNI, líneas, ofertas, escenarios de preevaluación).
2. Simulador de preevaluación y autorización adicional (B-04).
3. Configuración financiera versionada (TEA pizarra, seguro).
4. UI interna P-010–P-013 y login interno.
5. Cierre de sesión y gestión de la expiración de sesión en el cliente.
6. Scheduler o evaluación de expiraciones (simulación, oferta, preevaluación).
7. Endpoints de reconsulta y reintento (B-06).
8. Tareas Could para P-015 y P-016, o retirarlas de 09.

---

## 8. Calidad de la UI Spec

### 8.1 Lo que está bien

Los 7 flujos (F-01–F-07) cubren los recorridos Must. Las 13 pantallas Must tienen propósito, datos, acciones, reglas, wireframe y estados. Los tokens, los objetivos táctiles, los breakpoints y los criterios transversales de resiliencia («nunca inferir éxito desde caché») son sólidos, y los prototipos P-001–P-013 los respaldan.

### 8.2 Gaps

| # | Gap | Severidad | Corrección |
|---|---|---|---|
| U-01 | Sin tabla de **rutas Angular** (paths, parámetros, guards por rol). | Alta | Añadir 09 §5.2 «Rutas». |
| U-02 | Sin mapeo **pantalla → endpoints** (qué llama cada pantalla y con qué cabeceras). | Alta | Añadir una columna «API» a cada Screen Spec. |
| U-03 | Sin **catálogo de mensajes por código de error** (depende de PA-34): P-007 dice «motivo funcional permitido» sin listarlo. | Alta | Tabla `code → mensaje → acción` en 09 o en el OpenAPI. |
| U-04 | Falta el login interno y el flujo de cierre de sesión. | Alta | Añadir F-00 «Cierre de sesión» y P-017 (ver §1.2). |
| U-05 | No especifica **cuándo** se registra la evidencia de presentación (RN-253) ni los eventos KPI por pantalla (`OPPORTUNITY_VIEWED` en P-002, `SIMULATION_PARAMETERS_CHANGED` en P-004, `SCHEDULE_VIEWED` en P-005). | Media | Añadir el apartado «Eventos» a cada Screen Spec. |
| U-06 | P-014–P-016 no tienen Screen Spec ni wireframe, solo inventario. | Baja (Could) | Completar o marcar «fuera de v1». |
| U-07 | P-003 usa el estado «vencida», que no es canónico (depende de B-01). | Media | Alinear con el estado que se cree en 03. |
| U-08 | El versionado de 09 (0.2, «propuesta») no refleja la validación con prototipos, porque `10-validation-log.md` está vacío. | Media | Consolidar la validación (paso 11) y subir de versión. |

**¿Los wireframes son suficientemente detallados?** Sí para las pantallas Must: el orden de elementos, los rótulos, los estados y las variantes móviles alcanzan para que un agente construya las pantallas. No lo son en la integración: faltan rutas, llamadas a la API y textos de error (U-01–U-03).

---

## 9. Score general

| Criterio | Score | Justificación | Qué lo sube a 5 |
|---|:-:|---|---|
| **Operatividad** | 3 | Un agente puede implementar el dominio financiero y gran parte de contratación y desembolso, pero se detiene en B-01–B-07. | Resolver los bloqueantes, ADR-007–011 y las tareas faltantes. |
| **Utilidad** | 4 | Las RN en formato EARS, los criterios con valores límite, los vectores numéricos y la matriz canónica dan un valor de verificación muy alto. | Corregir el ejemplo de §4.5 y añadir el mapeo pantalla→API. |
| **Orden** | 3 | La estructura y la numeración son limpias, pero hay versiones dispares, el archivo 10 está vacío, la lista oficial de archivos está desactualizada, hay colisiones de IDs y mezcla de idiomas. | Unificar la versión 1.0, completar el 10, crear la tabla de correspondencia y renombrar los IDs QA. |
| **Global** | **3,3** | | |

---

## 10. Plan de corrección priorizado

1. **03, 04 y 05:** cerrar B-01 (estados y transiciones de Solicitud), B-08 (glosario) y registrar las PA de §5.2. Reclasificar PA-33 y PA-34.
2. **07:** corregir §4.4 y §4.7, añadir los endpoints de B-06, las entidades de B-07, el §4.13 de preevaluación simulada, ADR-007–011 y el ejemplo de §4.5.
3. **Política operativa:** documentar los límites de recuperación y reintento (B-03).
4. **08:** dividir las tareas no atómicas, añadir el campo «Pantallas», separar T-020a y crear las tareas de §7.4.
5. **09:** añadir rutas, mapeo API, catálogo de mensajes, login interno, cierre de sesión y eventos.
6. **00, 05 y encabezados:** declarar el Spec Pack v1.0 con sus 11 archivos y completar `10-validation-log.md`.
7. Volver a ejecutar esta auditoría. El criterio de salida es **0 bloqueantes** y Operatividad ≥ 4.
