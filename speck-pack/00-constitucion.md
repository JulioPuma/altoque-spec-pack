# Constitución del proyecto Altoque

**Archivo:** `00-constitucion.md`  
**Versión del Spec Pack:** 0.1  
**Estado:** Vigente y aprobada formalmente  
**Aprobación:** Equipo responsable del producto Altoque, versión 0.1  
**Naturaleza:** Producto web de crédito de consumo para el mercado peruano  
**Fuentes:** Base v0.1, auditoría QA y ocho rondas de refinamiento. Trazabilidad en [05-preguntas-abiertas.md](05-preguntas-abiertas.md).

La versión 0.1 es la Constitución vigente del proyecto. Su aprobación no convierte en aprobadas las decisiones que permanecen abiertas en `05-preguntas-abiertas.md`.

El Spec Pack oficial está compuesto por `00-constitucion.md`, `01-project-brief.md`, `02-alcance.md`, `03-reglas-de-negocio.md`, `04-glosario.md` y `05-preguntas-abiertas.md`. Los seis archivos DEBEN distribuirse y revisarse juntos. Las decisiones cerradas de `05-preguntas-abiertas.md` deben reflejarse en los archivos normativos afectados; una pregunta abierta bloquea únicamente las reglas o partes que la referencian y NO constituye una decisión aprobada.

Altoque es un producto web que busca agilizar la solicitud de créditos de consumo para personas en el Perú, incluidas quienes aún no son clientes. Los recorridos y capacidades habilitados en la primera evaluación se delimitan en el alcance. La primera evaluación se ejecuta en un entorno cerrado: solo acceden clientes sintéticos cuyas cuentas fueron creadas previamente para la evaluación. El registro autónomo de personas y el acceso con datos personales reales no forman parte de esta evaluación. Los clientes sintéticos con oferta preferencial pueden simularla y los clientes sintéticos sin ella siguen el flujo regular. Los criterios de elegibilidad y el grado de integración operativa requieren decisiones explícitas. Ningún principio convierte en aprobada una decisión que figure en `05-preguntas-abiertas.md`.

## PRINCIPIO-001: Código en inglés y documentación en español

Los identificadores y nombres del código DEBEN escribirse en inglés. Los valores de estado de dominio publicados o persistidos conservan el catálogo canónico de RN-069. La documentación funcional, técnica y arquitectónica DEBE escribirse en español.

**Cumple:** `LoanSimulationService` y su ADR en español.  
**Viola:** `ServicioSimulacionCredito` o documentación funcional solo en inglés.

## PRINCIPIO-002: Stack tecnológico obligatorio

El backend DEBE usar Java y Spring Boot y el frontend DEBE usar Angular. Las versiones exactas y compatibles DEBEN fijarse antes de implementar y cualquier sustitución de tecnología principal requiere ADR y aprobación humana.

## PRINCIPIO-003: Cálculo financiero centralizado

Cuotas, intereses, TCEA, costo total y cronograma DEBEN calcularse en un único módulo backend. El frontend NO DEBE mantener fórmulas financieras independientes.

## PRINCIPIO-004: Transparencia antes de aceptar

Antes de aceptar, el cliente DEBE visualizar monto, plazo, cuota financiera, seguro, pago total por cuota, intereses, TCEA, costo total y cronograma correspondientes a la misma simulación que acepta.

## PRINCIPIO-005: Correspondencia íntegra de condiciones

La aceptación DEBE identificar una versión completa e inmutable de las condiciones. La contratación DEBE utilizar exactamente esa aceptación vigente; un cambio financiero exige nueva simulación y nueva aceptación.

## PRINCIPIO-006: Autenticación, consentimiento y atomicidad contractual

Ningún contrato puede confirmarse sin cliente autenticado, aceptación expresa vigente y evidencia auditable. Antes del registro contractual, el intento y su reserva DEBEN existir de forma durable. El contrato, su auditoría obligatoria, la orden única de desembolso y la conversión de la reserva en consumo DEBEN confirmarse en una única transacción; si falla cualquiera de esos componentes, ninguno se confirma y no se declara contratación exitosa.

## PRINCIPIO-007: Unicidad y control de concurrencia

Aceptación, simulación y contratación DEBEN serializarse por oportunidad de crédito. Cada simulación puede conservar su propia aceptación vigente. Las repeticiones de la misma aceptación DEBEN deduplicarse por cliente, simulación y versión inmutable. Desde que la contratación adquiere el bloqueo de oportunidad se bloquean nuevas aceptaciones. Cuando una contratación requiere ambos bloqueos, DEBE adquirir primero la oportunidad y después la línea del cliente; ningún flujo puede invertir ese orden. Después de esperar un bloqueo se DEBEN revalidar estados, versiones y guardas aplicables. Solo puede existir un contrato por oferta preferencial o por solicitud regular. La línea del cliente es compartida por ambos recorridos. La reserva al registrar DEBE protegerse atómicamente frente a otras contrataciones de la misma línea y su conversión o liberación solo puede aplicarse una vez.

## PRINCIPIO-008: Desembolso único y resultado inequívoco

Una contratación NO DEBE producir más de un desembolso lógico. Un timeout o respuesta ambigua NO equivale a fallo ni a éxito. La orden de desembolso y su identificador persistente DEBEN confirmarse junto al contrato. Tras una interrupción, la recuperación DEBE consultar la operación original. Si la consulta no acredita un resultado terminal, el sistema puede reentregar técnicamente el comando original usando exactamente el mismo identificador persistente y los mismos datos; NO DEBE crear otro identificador ni ejecutar una segunda operación lógica. El core simulado DEBE deduplicar atómicamente por identificador y rechazar como conflicto el mismo identificador con datos distintos.

## PRINCIPIO-009: Máquinas de estados independientes

Solicitud, simulación, contratación y desembolso DEBEN mantener identidad y estado propios. Los estados terminales confirmados son inmutables; cualquier corrección requiere una nueva operación o un registro de ajuste. En solicitud, simulación y contratación, los estados `FALLIDA_RECUPERABLE` son no terminales y solo admiten un reintento controlado de la misma operación; los estados `FALLIDA_DEFINITIVA` son terminales. `DESEMBOLSO_FALLIDO` solo es terminal cuando cumple RN-053.

## PRINCIPIO-010: Auditoría y trazabilidad

Aceptaciones, intentos de contratación, contratos, desembolsos y cambios de estado DEBEN conservar identificadores y correlación. Como mínimo se registra operación, cliente, instante, condiciones o versión aplicable, resultado y motivo. Cada transición DEBE registrar operación, cliente, actor, fecha y hora, origen, destino, motivo, correlación y versión. Para confirmar una transición se DEBE validar el estado origen y la versión de entidad.

## PRINCIPIO-011: Calidad verificable

Backend y frontend DEBEN alcanzar por separado al menos 80 % de cobertura de líneas. Todo cambio de lógica requiere pruebas automatizadas. Antes de integrar se requiere SAST; los hallazgos deben resolverse o contar con aceptación de riesgo humana documentada.

## PRINCIPIO-012: Accesibilidad básica

Los flujos principales DEBEN poder operarse por teclado y sus controles DEBEN tener etiquetas identificables. WCAG 2.2 AA es la referencia de accesibilidad, incluidos sus criterios de contraste aplicables. La seguridad se verifica con OWASP ASVS 5.0.0 nivel 1. Antes de integrar no pueden permanecer hallazgos críticos o altos abiertos; un riesgo medio requiere aceptación humana documentada. Los controles, herramientas y evidencias detallados se documentan en el plan de seguridad.

## PRINCIPIO-013: Datos de verificación y límites de agentes

Para desarrollo, pruebas y ejemplos de la primera evaluación solo se usarán datos sintéticos. Los repositorios y artefactos de verificación NO DEBEN incorporar credenciales ni datos financieros personales reales. El uso operativo de datos reales queda sujeto a una decisión de alcance. Los agentes de IA pueden leer y editar archivos autorizados, pero NO pueden ejecutar código, pruebas, scripts o aplicaciones ni desplegar o publicar artefactos. Una persona responsable ejecuta las verificaciones del proyecto o autoriza su ejecución en CI y aporta sus resultados como evidencia. Esta asignación operativa aplica al proyecto y deriva de PRINCIPIO-011 y PRINCIPIO-013.

Las credenciales de las cuentas sintéticas DEBEN generarse y almacenarse separadamente de sus identificadores. Una contraseña NO DEBE ser igual al DNI sintético ni derivarse de él.

## PRINCIPIO-014: Gobernanza documental

La documentación se mantiene en Markdown con BMAD/SDD, las decisiones arquitectónicas en ADR y los contratos HTTP en OpenAPI. Los requisitos pendientes NO DEBEN presentarse como decisiones aprobadas. Modificar esta constitución requiere aprobación humana y cambio de versión.

## Invariantes aprobados

| ID | Invariante | Fuente |
|---|---|---|
| INV-01 | Cada simulación conserva su propia aceptación y las repeticiones de la misma aceptación se deduplican. | Ronda de refinamiento 5, sustituye DEC-55 para este invariante |
| INV-02 | La contratación bloquea aceptaciones concurrentes. | DEC-56 |
| INV-03 | Un contrato registrado es único por clave de negocio. | Rondas 6–9 / DEC-60 |
| INV-04 | Contrato y auditoría se confirman atómicamente. | Rondas 6–7 |
| INV-05 | No se ejecuta dos veces el desembolso de una contratación. | Rondas 6–7 |
| INV-06 | Un timeout no equivale a desembolso fallido. | Rondas 6–7 |
| INV-07 | Los estados terminales confirmados no se modifican. | DEC-61 |
| INV-08 | Los reintentos técnicos son controlados. | DEC-59, DEC-78 |
| INV-09 | Toda transición registra operación, cliente, actor, fecha y hora, origen, destino, motivo, correlación y versión. | DEC-71, DEC-94 |
| INV-10 | Toda transición valida estado origen y versión de entidad antes de confirmarse. | DEC-72 |
| INV-11 | El intento y su reserva se crean atómicamente antes del registro contractual. | Cierre de PA-28 |
| INV-12 | Contrato, auditoría contractual, orden única de desembolso y conversión de reserva en consumo se confirman en una sola transacción. | Cierre de PA-28 |
| INV-13 | La recuperación contractual se resuelve únicamente mediante evidencia durable; la ausencia de respuesta o de lectura concluyente conserva el intento y la reserva pendientes. Tras 15 minutos sin resolución automática, el intento pasa a revisión manual contractual con objetivo de resolución de 4 horas; el tiempo por sí solo nunca acredita un resultado. | Cierre de PA-28 y PA-37 |
| INV-14 | Solo una orden durable en `DESEMBOLSO_SOLICITADO`, adquirida exclusivamente, puede iniciar el despacho. Desde `DESEMBOLSO_PROCESANDO`, la recuperación consulta la operación original y puede reentregar idempotentemente el mismo comando, con idéntico identificador y datos; nunca crea ni ejecuta una segunda operación lógica. | Cierre de PA-26 |
| INV-15 | Los bloqueos se adquieren siempre en orden oportunidad–línea y toda operación revalida sus guardas después de esperar. | Cierre de PA-11 |
| INV-16 | Una aceptación se deduplica por cliente, simulación y versión inmutable; la conversión o liberación de una reserva solo produce efecto una vez. | Cierre de PA-11 |
| INV-17 | Los resultados terminales del core simulado son inmutables; una respuesta contradictoria no sobrescribe un terminal local confirmado y se escala como conflicto. | Cierre de PA-04 |
| INV-18 | La preevaluación y su eventual autorización son identificadas, versionadas, auditables y válidas durante 24 horas con límite exclusivo UTC; datos obligatorios faltantes nunca se completan por defecto. | Cierre de PA-27 |
| INV-19 | Cada emisión preferencial y cada solicitud regular nueva conservan identidad propia e inmutable; reemplazo, vencimiento, contratación o cierre no reutilizan esa identidad. | Cierre de PA-01 |

## Gobierno de cambios

1. La constitución prevalece sobre instrucciones posteriores.
2. Una contradicción obliga a detener la acción afectada y registrar el conflicto.
3. Una decisión ausente se traslada a `05-preguntas-abiertas.md`; no se completa por inferencia.
4. Los ejemplos no fijan parámetros de negocio.
