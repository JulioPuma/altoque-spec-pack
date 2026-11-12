# Altoque — Definición de alcance

**Archivo:** `02-alcance.md`  
**Versión:** 0.1  
**Estado:** Alcance funcional consolidado

La primera evaluación se realiza en un entorno cerrado con datos sintéticos y cuentas creadas previamente. No incluye registro autónomo, alta de personas reales ni integración con servicios externos de identidad. El cliente sintético con oferta preferencial puede simularla. El cliente sintético sin oferta preferencial sigue el flujo regular, que puede requerir evaluación y habilitar una oferta de monto reducido. Se asume que existe una preevaluación.

## 1. Alcance incluido — MUST

| ID | Capacidad | Criterio de aceptación resumido |
|---|---|---|
| M-01 | Autenticación y acceso propio | Solo una cuenta sintética creada previamente puede autenticarse en la primera evaluación. Usa un DNI sintético como identificador y una contraseña sintética independiente, no igual ni derivada del DNI. Un cliente solo accede a sus recursos. |
| M-02 | Inicio de solicitud | El cliente elige explícitamente iniciar el flujo regular, se crea un `solicitudId` y luego se obtiene la preevaluación identificada y versionada asociada. Dura 24 horas y puede habilitar una oferta base, exigir evaluación adicional, confirmar ausencia de oferta o resultar no disponible. Solo `SOLICITUD_HABILITADA` permite simular. |
| M-03 | Simulación financiera backend | Publica como exitosa una simulación completa, válida y almacenada. |
| M-04 | Presentación y aceptación expresa | Muestra condiciones completas y conserva evidencia. |
| M-05 | Contratación única y auditable | Intento y reserva persistidos atómicamente antes del registro; contrato, auditoría, orden de desembolso y conversión de reserva confirmados en una única transacción; un único contrato registrado. |
| M-06 | Desembolso simulado idempotente | Un despachador adquiere exclusivamente la orden durable e inicia el despacho desde `DESEMBOLSO_SOLICITADO`. La recuperación puede reentregar el mismo comando con idéntico identificador y datos; el core deduplica y ejecuta una sola operación lógica. |
| M-07 | Recuperación del resultado | Consulta la operación original y escala a revisión manual si la recuperación automática no resuelve el resultado. |
| M-08 | Cuatro máquinas de estados | Solicitud, simulación, contratación y desembolso independientes. |
| M-09 | Instrumentación de KPIs | Conserva datos necesarios sin inventar reglas pendientes. |
| M-10 | Consulta posterior de contratación y desembolso | Al volver a ingresar, el cliente puede consultar sus operaciones propias, pendientes y resueltas. |

## 2. Should / Could

| ID | Prioridad | Capacidad | Límite |
|---|---|---|---|
| S-01 | SHOULD | Comunicación ampliada de vencimientos | No autoriza contratar entidades vencidas. |
| S-03 | SHOULD | Gestión visible de fallas y revisión | No cambia estados finales sin evidencia. |
| S-04 | SHOULD | Indicadores complementarios | Solo tras aprobar sus definiciones. |
| C-01 | COULD | Comparación de simulaciones | No convierte una comparación en aceptación. |
| C-02 | COULD | Ayuda financiera contextual | No sustituye el desglose obligatorio. |
| C-03 | COULD | Historial de simulaciones | Respeta pertenencia y vigencia. |

## 3. Fuera de alcance

- Integraciones productivas y movimientos de dinero reales durante la primera evaluación.
- Scoring o evaluación crediticia real.
- Créditos hipotecarios, vehiculares o empresariales.
- Cobranza, pago de cuotas, refinanciamiento y operaciones posteriores de negocio. Los ajustes técnicos de RN-033 tienen identidad propia y preservan el terminal confirmado.
- Aplicaciones móviles nativas, chatbot, recomendaciones y notificaciones externas.
- Firma digital o biometría de proveedores no aprobados.
- Registro autónomo, alta de personas reales e integración con RENIEC u otro servicio externo de identidad.
- Panel administrativo bancario completo.
- Declaración de cumplimiento regulatorio productivo.

## 4. Responsabilidades por máquina

| Máquina | Responsabilidad | No implica |
|---|---|---|
| Solicitud | Identificar y habilitar el recorrido | Simulación ni elegibilidad crediticia real |
| Simulación | Calcular una versión de condiciones | Aceptación o contrato |
| Contratación | Validar aceptación e intentar registrar contrato y auditoría | Desembolso confirmado |
| Desembolso | Ejecutar una vez, consultar y reconciliar | Reejecutar ante timeout |

Una solicitud conserva sus simulaciones previas. «Aceptar» registra la conformidad. «Contratar» inicia un intento independiente. Una simulación nueva no invalida por sí sola otra simulación ni su aceptación. Cada simulación conserva su vigencia y requiere su propia aceptación. La aceptación selecciona una versión. Cada intento tiene identidad propia. Existe un contrato como máximo por oferta o solicitud, según el recorrido. Cada emisión preferencial tiene un `ofertaId` inmutable y cada solicitud regular nueva un `solicitudId` inmutable. Una identidad cerrada no se reutiliza. El desembolso tiene identificador persistente y único asociado al contrato.

## 5. Modelo financiero aprobado

- Sistema francés de cuota financiera.
- TEA de oferta o de configuración global para tasa pizarra.
- Tasa mensual: `i_m = (1 + TEA)^(1/12) - 1`.
- Cuota financiera: `C = P × i_m / (1 - (1 + i_m)^(-n))`.
- Seguro fijo global sumado a cada cuota.
- Moneda de cálculo de la primera evaluación: PEN. Cálculo con `BigDecimal`, precisión interna de 34 dígitos y `HALF_UP`. Importes monetarios a dos decimales con `HALF_UP`.
- Con TEA cero, `C = P/n`.
- Períodos financieros mensuales uniformes. Primera cuota al mes siguiente con ajuste de fin de mes.
- Última cuota ajustada para cancelar exactamente el saldo.
- TCEA calculada con flujos de desembolso y cuotas que incluyen seguro, anualizando la TIR mensual.

Configuración normativa inicial de la primera evaluación:

- Moneda única PEN.
- Monto solicitado desde S/ 500 hasta S/ 10 000 inclusive, en incrementos de S/ 100, sin superar el máximo de la oferta ni el saldo disponible.
- Plazos permitidos de 6, 12, 18 o 24 meses.
- TEA válida entre `0.00` y `1.00` inclusive; TEA pizarra inicial `0.20`.
- Seguro fijo inicial de S/ 10 por cuota, configurable entre S/ 0 y S/ 100 inclusive.
- Cada cuenta sintética contiene obligatoriamente límite de línea, moneda PEN, consumo actual, reservas vigentes y versión de línea. El límite debe ser mayor que S/ 0 y no superar S/ 50 000.
- La ausencia de un dato obligatorio bloquea la simulación o contratación afectada y nunca se completa con un valor predeterminado.
- La tasa y el seguro quedan congelados en la versión de simulación. Un cambio de configuración solo afecta simulaciones nuevas. El límite, saldo y versión de línea se revalidan al crear la reserva y pueden impedir contratar una simulación anterior.

La TEA se representa como decimal: 20 % se almacena como `0.20`. La tasa mensual se obtiene resolviendo `(1 + i_m)^12 = 1 + TEA` con precisión interna de 34 dígitos y se redondea a 16 decimales con `HALF_UP`. La cuota francesa teórica se calcula con precisión interna y se redondea a dos decimales. En cada período, el interés es el saldo inicial multiplicado por la tasa mensual, redondeado a dos decimales; el capital es la cuota financiera menos el interés y el saldo final es el saldo inicial menos el capital. En la última cuota, el capital es el saldo pendiente, el interés se calcula sobre ese saldo y la cuota financiera se ajusta a capital más interés. El seguro fijo no se modifica por este ajuste.

La TCEA usa el desembolso inicial positivo y los pagos mensuales realmente redondeados del cronograma, incluido el seguro, como flujos negativos. La TIR mensual se obtiene mediante bisección determinista con tolerancia `1E-12`, máximo de 1 000 iteraciones y dominio mayor que `-1`. La búsqueda usa el intervalo inicial `[-0.999999999999, 1000000]`; si no existe cambio de signo o no converge, la simulación falla con error explícito. La TCEA es `(1 + TIR_mensual)^12 - 1`, se conserva internamente con 12 decimales y se presenta como porcentaje con dos decimales, usando `HALF_UP`. El costo total es la suma de pagos del cronograma con seguro incluido.

La fecha de creación de la simulación es el ancla del cronograma. La primera cuota vence el mismo número de día del mes siguiente; si ese día no existe, vence el último día de ese mes. Si el ancla es el último día de su mes, todas las cuotas vencen el último día de sus meses. Los períodos financieros y de TCEA son mensuales uniformes y las diferencias de días no generan prorrateo.

Vectores normativos mínimos:

| Caso | Entrada | Resultado obligatorio |
|---|---|---|
| TEA cero y una cuota | `P=1000.00`, `TEA=0`, `n=1`, seguro `10.00` | Interés `0.00`, capital `1000.00`, cuota financiera `1000.00`, pago `1010.00`, costo total `1010.00`, TCEA presentada `12.68 %`. |
| Ajuste final con TEA cero | `P=1000.00`, `TEA=0`, `n=3`, seguro `0.00` | Capitales `333.33`, `333.33`, `333.34`; saldo final `0.00`; TCEA presentada `0.00 %`. |
| Ancla día 31 | Creación `2026-01-31`, tres cuotas | Vencimientos `2026-02-28`, `2026-03-31`, `2026-04-30`. |
| Año bisiesto | Creación `2028-01-30`, dos cuotas | Vencimientos `2028-02-29`, `2028-03-30`. |
| TIR sin solución | Flujo técnico sin cambio de signo dentro del dominio | `SIMULACION_FALLIDA_DEFINITIVA` y error explícito; no se publica TCEA. |

Los valores y rangos configurables para la primera evaluación son los definidos en esta sección y en RN-195–202.

## 6. Dependencias

| ID | Dependencia | Estado |
|---|---|---|
| DEP-01 | Datos sintéticos de clientes y ofertas | Requerida; invariantes y campos obligatorios aprobados en RN-195–202. |
| DEP-02 | Autenticación | Cuentas sintéticas precargadas, contraseñas independientes de sus identificadores y roles funcionales aprobados. El mecanismo técnico se documenta en la especificación de seguridad. |
| DEP-03 | Módulo financiero | Requerida. |
| DEP-04 | Core simulado con consulta por operación | Requerida; catálogo funcional aprobado en RN-144–153 y contrato técnico documentado en OpenAPI. |
| DEP-05 | Persistencia transaccional y auditoría | Requerida. |
| DEP-06 | Instrumentación de eventos | Requerida; catálogo pendiente. |

## 7. Criterios de aceptación transversales

1. Dada una oferta ajena, cuando un cliente la consulta, entonces no se revela su contenido.
2. Dada una simulación modificada, cuando el cliente intenta contratar con aceptación anterior, entonces se rechaza y se exige nueva aceptación.
3. Dadas dos simulaciones vigentes de la misma oportunidad, si sus aceptaciones se confirman antes de contratar, ambas se conservan. Repetir una misma aceptación devuelve la existente.
4. Dada una contratación y una aceptación concurrentes, cuando la contratación adquiere el bloqueo, entonces se impide confirmar una nueva aceptación.
5. Dadas solicitudes concurrentes de desembolso, cuando comparten contratación, entonces solo una ejecución puede iniciarse.
6. Dado un timeout, cuando se recupera, entonces se consulta la operación original sin reejecutarla.
7. Dado un estado terminal confirmado, cuando se solicita corregirlo, entonces se crea nueva operación o ajuste; no se sobrescribe.

8. Dada una oferta preferencial propia vigente, al verificar en `emisión + 30 días calendario` UTC, se considera vencida. Si vence entre la aceptación y el registro, se exige un recorrido vigente.
9. Dada una simulación exitosa, al verificar en `creación + 30 minutos` UTC, se considera vencida. Generar otra conserva la anterior y no invalida por sí solo su aceptación.
10. Dada una solicitud regular habilitada, al simular se aplica la TEA de configuración y se permite como máximo un contrato por solicitud. Los valores configurables están definidos en 02 §5 y RN-195–202; el contrato funcional de preevaluación está en RN-154–166.
11. Dado un commit incierto, al solicitar otro intento se consulta evidencia. Si el contrato ya existe, se devuelve al cliente autorizado. Si persiste la incertidumbre, el intento sigue pendiente.

12. Dado un cliente con S/ 10 000 disponibles, cuando dos intentos por oportunidades distintas reservan S/ 6 000 cada uno, entonces solo uno puede reservar y continuar al registro. Aplica aunque los intentos pertenezcan a recorridos distintos.
13. Dada una reserva cuyo intento queda incierto o en fallo recuperable, el monto sigue indisponible. Si se confirma el contrato, se convierte en consumo. Si el intento termina en `CONTRATACION_RECHAZADA` o `CONTRATACION_FALLIDA_DEFINITIVA`, se libera exactamente una vez.
14. Dado un contrato confirmado, una segunda simulación asociada a la misma oferta preferencial o solicitud regular no puede originar otro contrato para esa oportunidad. Un nuevo préstamo exige otra oportunidad y saldo disponible.

15. Dada una oferta base habilitada, el sistema aplica el máximo de la preevaluación y la TEA pizarra configurada. Al contratar, valida también el saldo disponible de la línea.
16. Dado un contrato confirmado, su orden de desembolso ya existe con identificador persistente. Tras una caída, el sistema solo retoma el envío si puede demostrar que nunca comenzó. Si pudo comenzar, consulta el original.
17. Dado un desembolso fallido de forma definitiva, se conservan el contrato y el consumo de línea. Una eventual liberación requiere un ajuste autorizado y auditable.

18. Dada una simulación presentada y vigente, cuando el cliente la acepta, se registra la evidencia de la acción expresa y la versión inmutable mostrada. Contratar usa exactamente esa aceptación.
19. Dada una respuesta de registro perdida, se consulta el intento ya persistido cada 30 segundos durante un máximo de 15 minutos y se conserva la reserva. Si no se resuelve, pasa a revisión contractual manual con objetivo de resolución de 4 horas. Una segunda contratación para la misma oportunidad queda bloqueada hasta confirmar el contrato o el cierre sin contrato; el tiempo transcurrido por sí solo nunca libera la reserva.
20. Dada una preevaluación sin respuesta, se informa indisponibilidad y se admite otra consulta. Dada una respuesta confirmada sin oferta, se comunica el resultado y se bloquean simulación y contratación en ese recorrido.

21. Dada una aceptación sobre una oferta aplicable y una simulación vigentes, deja de autorizar una contratación en el primero de sus vencimientos o al invalidarse la versión aceptada. Se mantienen los límites de 30 días de la oferta preferencial, 24 horas de la oferta base ligada a su preevaluación y 30 minutos de la simulación.
22. Si el mismo cliente repite una contratación con la misma clave y datos, recibe el intento original con su estado actual. Datos distintos producen conflicto. Una clave nueva no evita las guardas de unicidad, intento incierto y saldo.
23. Si el cliente cierra sesión con un resultado pendiente, al volver a ingresar puede consultar su estado actualizado. La consulta de una operación ajena no revela su contenido.
24. Dado un nuevo intento permitido, cuando se prepara el registro contractual, entonces la identidad del intento y su reserva se confirman atómicamente antes de registrar el contrato.
25. Dado un intento con reserva vigente, cuando se confirma el registro contractual, entonces contrato, auditoría obligatoria, orden única de desembolso y conversión de la reserva en consumo se confirman en una sola transacción. El fallo de cualquiera revierte el conjunto.
26. Dada una respuesta contractual perdida, cuando se recupera el intento, entonces solo se declara éxito si la lectura durable confirma el conjunto contractual. Solo se cierra sin contrato y se libera la reserva si existe evidencia durable de rollback o cierre; cualquier lectura inconclusa conserva `CONTRATACION_PENDIENTE` y la reserva.
27. Dada una orden en `DESEMBOLSO_SOLICITADO`, cuando un despachador la adquiere exclusivamente, entonces confirma durablemente `DESEMBOLSO_PROCESANDO` antes de realizar la primera entrega del comando.
28. Dada una interrupción, cuando la orden permanece en `DESEMBOLSO_SOLICITADO` sin una adquisición confirmada, entonces otro despachador puede adquirirla. Cuando la orden alcanzó `DESEMBOLSO_PROCESANDO`, la recuperación consulta la operación original y puede reentregar el comando únicamente con el mismo `operacionId` y datos.
29. Dada una consulta cuyo resultado es “operación no encontrada”, el desembolso permanece pendiente y puede reentregarse idempotentemente el comando original. Una recepción repetida idéntica devuelve la operación original; el mismo identificador con datos distintos produce conflicto sin ejecución.
30. Dada una contratación que necesita bloquear oportunidad y línea, cuando adquiere sus bloqueos, entonces siempre bloquea primero la oportunidad y después la línea. Ningún flujo usa el orden inverso.
31. Dada una operación que esperó un bloqueo, antes de confirmar cualquier efecto revalida estado y versión de la oportunidad, aceptación y vigencias aplicables, ausencia de contrato o intento incierto, y saldo y versión de la línea.
32. Dada una aceptación que llega mientras la contratación posee el bloqueo de oportunidad, espera y se revalida al continuar. Se rechaza si ya existe contrato; puede continuar si la contratación cerró sin contrato y la simulación permanece vigente.
33. Dadas dos contrataciones de oportunidades distintas del mismo cliente, cuando compiten por saldo, se serializan mediante la línea compartida y solo reservan sobre el saldo revalidado.
34. Dadas aceptaciones concurrentes con igual cliente, simulación y versión inmutable, cuando se confirman, se conserva un solo registro y todas las repeticiones reciben esa aceptación.
35. Dada una reserva, cuando procesos concurrentes intentan convertirla o liberarla, el control de estado y versión permite que solo una transición produzca efecto.
36. Dada una respuesta `EXITOSA`, `RECHAZADA` o `FALLIDA_DEFINITIVA` del core simulado, cuando su persistencia local se confirma, el desembolso pasa respectivamente a `DESEMBOLSO_SIMULADO_EXITOSO`, `DESEMBOLSO_RECHAZADO` o `DESEMBOLSO_FALLIDO`.
37. Dada una respuesta `RECIBIDA`, `PROCESANDO` o `NO_ENCONTRADA`, o un timeout, desconexión, respuesta inválida o código desconocido, el resultado permanece pendiente y se consulta la operación original.
38. Dado un rechazo descubierto mediante consulta, se confirma `DESEMBOLSO_RECHAZADO`. Dada una respuesta tardía, solo puede resolver un estado local pendiente.
39. Dada una respuesta que contradice un terminal local confirmado, no se modifica el terminal: se registra el conflicto con ambas evidencias y se escala a revisión manual.
40. Dado que el cliente inicia explícitamente el flujo regular, se crea primero un `solicitudId` en `SOLICITUD_INICIADA`, pasa a `SOLICITUD_VALIDANDO` y se solicita la preevaluación asociada. La respuesta confirmada contiene `resultadoId`, cliente, solicitud, versión, estado, `creadoEn` y `venceEn`; vence 24 horas después de su creación con límite exclusivo UTC.
41. Dada `OFERTA_BASE_HABILITADA`, contiene monto máximo y moneda y permite simular con la TEA pizarra. Dada `EVALUACION_ADICIONAL_REQUERIDA`, contiene `evaluacionId` y bloquea contratar hasta una autorización vigente.
42. Dada una autorización `PENDIENTE`, la solicitud permanece en `SOLICITUD_ESPERANDO_AUTORIZACION` y se permite consultar. Dada `DENEGADA`, pasa a `SOLICITUD_RECHAZADA`. Dada `AUTORIZADA`, pasa a `SOLICITUD_HABILITADA` hasta el vencimiento del resultado asociado.
43. Dada una respuesta `SIN_OFERTA`, se bloquean simulación y contratación de esa solicitud. Dada `NO_DISPONIBLE`, se informa indisponibilidad y se permite reconsultar sin inferir ausencia de oferta.
44. Dada una respuesta con datos obligatorios faltantes, se trata como `NO_DISPONIBLE` y no se completa ningún valor por defecto.
45. Dada una reconsulta concluyente que cambia estado, máximo o autorización, se crea una nueva versión auditada que reemplaza la versión activa e invalida para contratación sus simulaciones y aceptaciones. `NO_DISPONIBLE` no reemplaza una versión concluyente todavía vigente.
46. Dada una nueva emisión o reemplazo de oferta preferencial, se crea un `ofertaId` nuevo y se conserva la emisión anterior en historial. Una oferta vencida, reemplazada o contratada no admite nuevas simulaciones ni contrataciones.
47. Dadas varias ofertas preferenciales vigentes, el cliente elige explícitamente una. Cada `ofertaId` constituye una oportunidad distinta, admite como máximo un contrato y compite por la línea compartida.
48. Dado que una oferta preferencial vence, se reemplaza o se contrata, el sistema no inicia automáticamente el flujo regular. Este solo se habilita mediante una preevaluación vigente.
49. Dado un cliente, puede tener como máximo una solicitud regular activa. Reconsultar la preevaluación crea una versión dentro de esa solicitud y no otra oportunidad.
50. Dada una solicitud regular con contrato, queda cerrada. Dada una solicitud cerrada sin contrato, tampoco se reutiliza. Un nuevo préstamo requiere otro `solicitudId` y una nueva preevaluación.
51. Dado un error recuperable, solo se reintenta de forma controlada la misma operación. Dado un error definitivo, no se reintenta. Dado un resultado incierto, permanece pendiente hasta obtener evidencia concluyente.
52. Dado un desembolso en revisión manual, el operador autorizado puede consultar evidencia y registrar observaciones, pero no crear otra operación ni modificar un terminal sin evidencia concluyente.
53. Dado un cliente, operador, auditor o proceso automático, cada actor solo puede ejecutar las acciones autorizadas para su rol y ámbito. Una operación sobre un recurso ajeno no revela su contenido.
54. Dada una contratación repetida con la misma clave y datos, se devuelve el intento original. La misma clave con datos distintos produce conflicto sin ejecución; una clave nueva no evita las guardas vigentes.
55. Dada una simulación presentada, la aceptación solo se confirma si corresponde a la misma versión íntegra, propia, exitosa, vigente y no invalidada que estuvo disponible para el cliente.
56. Antes de integrar, la verificación de seguridad no presenta hallazgos críticos ni altos abiertos. Un riesgo medio solo continúa con aceptación humana documentada.

## 8. Control del alcance

Ninguna propuesta técnica es requisito aprobado hasta registrarse como decisión. Las preguntas de `05-preguntas-abiertas.md` condicionan únicamente las partes indicadas; no amplían el alcance.
