# Altoque — Glosario y lenguaje ubicuo

**Archivo:** `04-glosario.md`  
**Versión:** 0.1

| Término | Definición normativa |
|---|---|
| Aceptación | Confirmación expresa de una versión completa de condiciones simuladas. No es contrato. |
| Aceptación vigente | Aceptación cuya versión sigue válida hasta el primer vencimiento aplicable de la oferta o de la simulación. |
| Aceptación concurrente | Aceptación que coincide en el tiempo con otra. Las de simulaciones distintas conservan identidad propia. Se considera repetición la que comparte cliente, simulación y versión inmutable; no duplica su registro. |
| Ajuste | Registro separado que corrige efectos sin modificar un estado terminal confirmado. |
| Público objetivo | Personas en el Perú que necesitan solicitar un crédito de consumo, incluidas quienes aún no son clientes. |
| Persona nueva | Persona del público objetivo que aún no es cliente. Su registro autónomo no forma parte de la primera evaluación. |
| Flujo regular | Recorrido para clientes sin oferta preferencial. Puede requerir evaluación y habilitar préstamos de monto reducido. |
| Oferta base | Oferta de crédito en PEN del flujo regular, limitada por el máximo informado por la preevaluación y por RN-195–202, que aplica la TEA pizarra configurada y vence junto con la versión de preevaluación que la habilitó. |
| Cliente | Persona con una cuenta en Altoque. En la primera evaluación es una identidad sintética con cuenta creada previamente. |
| Cuenta sintética | Cuenta creada previamente para la primera evaluación, sin datos personales reales y con credenciales sintéticas independientes de su identificador. |
| Entorno cerrado | Entorno de la primera evaluación accesible solo mediante cuentas sintéticas creadas previamente; no admite registro autónomo ni personas reales. |
| Oferta preferencial | Emisión de condiciones preferenciales asignada a un cliente e identificada por un `ofertaId` inmutable. Reemplazarla crea otra emisión y conserva la anterior en historial. |
| Línea de crédito | Límite único en PEN del cliente, compartido por los recorridos preferencial y regular. En la primera evaluación, su límite, consumo, reservas y versión son datos sintéticos obligatorios. |
| Saldo disponible | Monto de la línea aún utilizable tras considerar el consumo y las reservas vigentes. |
| Reserva de crédito | Monto apartado atómicamente junto con la creación de un intento contractual. Se mantiene ante incertidumbre o fallo recuperable, se convierte en consumo dentro del conjunto contractual si se confirma el contrato y se libera exactamente una vez al confirmar `CONTRATACION_RECHAZADA` o `CONTRATACION_FALLIDA_DEFINITIVA`. |
| Preevaluación | Resultado identificado, versionado y auditable asociado al cliente y a una solicitud regular. Puede ser `OFERTA_BASE_HABILITADA`, `EVALUACION_ADICIONAL_REQUERIDA`, `SIN_OFERTA` o `NO_DISPONIBLE`. Una respuesta confirmada vence 24 horas después de `creadoEn`, con límite exclusivo UTC. |
| Autorización adicional | Resultado asociado a un `evaluacionId`, cliente y solicitud regular. Puede estar `PENDIENTE`, `AUTORIZADA` o `DENEGADA` y vence junto con la versión de preevaluación que la originó. |
| OFERTA_BASE_HABILITADA | Estado de preevaluación que contiene monto máximo positivo y moneda y permite simular aplicando la TEA pizarra configurada. |
| EVALUACION_ADICIONAL_REQUERIDA | Estado de preevaluación que contiene `evaluacionId` y bloquea la contratación hasta recibir autorización vigente. |
| SIN_OFERTA | Estado confirmado de preevaluación que bloquea simulación y contratación de la solicitud regular correspondiente. |
| NO_DISPONIBLE | Resultado inconcluso de preevaluación por timeout, desconexión, respuesta inválida o datos obligatorios faltantes. Permite reconsulta y no equivale a `SIN_OFERTA`. |
| Condiciones aceptadas | Versión inmutable de monto, plazo, tasa, cuota financiera, seguro, pago total por cuota, intereses, TCEA, costo total y cronograma aceptada por el cliente. |
| Contratación | Proceso que valida una aceptación y registra atómicamente contrato y auditoría. |
| Contrato | Registro único resultante de una contratación exitosa. |
| Costo total | Suma de los pagos del cronograma, cada uno con su seguro fijo incluido (RN-084). |
| Crédito preaprobado | Término histórico de la v0.1 para la oferta preferencial. Cada emisión vigente se acredita mediante su `ofertaId` conforme a RN-167–170. |
| Cuota financiera | Pago periódico calculado por sistema francés antes del seguro. |
| Desembolso simulado | Operación de la primera evaluación asociada al contrato registrado y ejecutada en un entorno simulado, sin movimiento de fondos. |
| Adquisición de orden | Bloqueo o leasing durable y exclusivo mediante el cual un único despachador obtiene el derecho de realizar la primera entrega de una orden en `DESEMBOLSO_SOLICITADO`. |
| Despachador de desembolso | Proceso automático que adquiere una orden durable, confirma `DESEMBOLSO_PROCESANDO` y realiza la primera entrega del comando al core simulado. |
| Core simulado | Componente de la primera evaluación que recibe una orden identificada de desembolso sin mover fondos reales, deduplica atómicamente por `operacionId`, conserva un único resultado por operación y permite consultarlo. Una repetición con idénticos datos devuelve la operación original; datos distintos para el mismo identificador producen conflicto sin ejecución. |
| Reentrega idempotente | Repetición técnica del comando original con idéntico `operacionId` y datos. Puede generar otra llamada, pero no otra operación lógica ni otro desembolso simulado. |
| Resultado inconcluso del core | `RECIBIDA`, `PROCESANDO`, `NO_ENCONTRADA`, timeout, desconexión, respuesta inválida o código desconocido. No acredita éxito, rechazo ni fallo definitivo. Permite consultar y, conforme a la política de recuperación, reentregar idempotentemente el comando original. |
| Resultado terminal del core | `EXITOSA`, `RECHAZADA` o `FALLIDA_DEFINITIVA`. Es inmutable y debe persistirse localmente antes de exponer el estado terminal correspondiente. |
| Conflicto de resultado | Respuesta del core que contradice un estado terminal local confirmado. Se audita y escala a revisión manual sin modificar el terminal. |
| Error recuperable | Error técnico confirmado que permite reintentar de forma controlada la misma operación dentro de los límites aprobados. |
| Error definitivo | Error respaldado por evidencia concluyente que no admite reintento automático de la misma operación. |
| Resultado incierto | Resultado para el que no existe evidencia concluyente de éxito, rechazo o fallo; permanece pendiente aunque se agoten consultas o reintentos. |
| Estado terminal confirmado | Estado respaldado por evidencia concluyente que no puede sobrescribirse. |
| CONTRATACION_FALLIDA_RECUPERABLE | Fallo técnico confirmado no terminal que conserva la reserva y permite únicamente un reintento controlado del mismo intento. |
| CONTRATACION_FALLIDA_DEFINITIVA | Fallo técnico confirmado terminal que no produjo contrato y libera la reserva exactamente una vez. |
| CONTRATACION_EN_REVISION_MANUAL | Estado no terminal de un intento contractual que no obtuvo evidencia concluyente durante los 15 minutos de recuperación automática. Conserva la reserva y bloquea otro intento hasta resolverse mediante evidencia durable. |
| FALLIDA/FALLIDO | Resultado de error usado por máquinas distintas de contratación; solo es recuperable si la clasificación aprobada así lo permite. |
| Intento de contratación | Operación identificada y con estado propio; varios intentos no permiten varios contratos. |
| Conjunto contractual | Unidad atómica formada por contrato, auditoría contractual obligatoria, orden única de desembolso y conversión de la reserva en consumo. Se confirma o revierte completa. |
| Evidencia durable contractual | Lectura autoritativa de datos confirmados que acredita el conjunto contractual o el cierre sin contrato. Una ausencia de respuesta o una lectura inconclusa no constituye evidencia de cierre. |
| Evidencia concluyente contractual | Conjunto contractual atómico completo, marcador durable de rollback o cierre sin contrato, o resultado autoritativo del gestor transaccional que pruebe el rollback. El tiempo transcurrido, la ausencia de filas, un timeout o una lectura no disponible no son evidencia concluyente. |
| Revisión manual contractual | Evaluación de un intento contractual incierto después de 15 minutos de recuperación automática. Tiene objetivo de resolución de 4 horas; si no obtiene evidencia concluyente, conserva la reserva y escala el incidente crítico. |
| Operación original | Operación persistida cuyo resultado se recupera mediante su identificador. En desembolso, la orden se identifica antes del envío y, una vez en `DESEMBOLSO_PROCESANDO`, solo puede consultarse, no reenviarse. |
| Oferta | Condiciones de crédito identificadas para un cliente. Puede ser preferencial o base. |
| Oferta única visualizada | Unidad de medición deduplicada por oferta y cliente; la primera visualización asigna la cohorte. El denominador y el seguimiento exacto están en PA-05. |
| PENDIENTE | Abreviatura descriptiva para un resultado sin evidencia concluyente. |
| PENDIENTE_DE_GESTION | Abreviatura de DESEMBOLSO_PENDIENTE_DE_GESTION. |
| Reintento controlado | Recuperación de la misma operación bajo la política aplicable. Para desembolso se limita a consulta, reentrega idempotente del mismo comando o persistencia del resultado original (RN-055). |
| Revisión manual | Evaluación del procesamiento original que solo puede cerrar con evidencia concluyente. Si falta evidencia, el resultado sigue pendiente. |
| CLIENTE_SINTETICO | Rol de una cuenta sintética que solo puede consultar y operar sus propios recursos de crédito. |
| OPERADOR_REVISION_DESEMBOLSO | Rol autorizado para consultar evidencia y registrar actuaciones sobre desembolsos en revisión, sin crear otra operación ni modificar terminales sin evidencia. |
| AUDITOR | Rol de consulta de estados y registros de auditoría autorizados, sin facultad para producir transiciones de negocio. |
| SERVICIO_INTERNO | Rol de un proceso automático limitado a las acciones expresamente asignadas a su función. |
| Evidencia de presentación | Registro auditable que acredita que la versión íntegra de una simulación y todas sus condiciones obligatorias estuvieron disponibles para el cliente antes de su aceptación. |
| Expediente de conflicto | Registro separado que conserva evidencias contradictorias sobre una operación terminal sin modificar su estado confirmado. |
| Seguro fijo | Importe de configuración global sumado a cada cuota financiera. |
| Simulación | Cálculo identificado de condiciones financieras; puede haber varias por solicitud. |
| Simulación exitosa | Cálculo completo, válido y almacenado; no implica aceptación ni contrato y puede expirar o invalidarse. |
| Solicitud | Recorrido identificado. En el flujo regular, un `solicitudId` identifica una oportunidad inmutable; solo puede existir una solicitud regular activa por cliente y una solicitud cerrada no se reutiliza. |
| Sistema francés | Método de cuota financiera periódica constante a dos decimales, sujeto al ajuste de la cuota final para cancelar el saldo conforme a RN-188–189. |
| Tasa pizarra | TEA configurada aplicada al flujo regular cuando la preevaluación habilita una oferta base. |
| TCEA | Tasa de costo efectivo anual obtenida por TIR mensual de los flujos redondeados de desembolso y pagos con seguro, anualizada conforme a RN-190–192. |
| TEA | Tasa efectiva anual de la oferta o configuración pizarra, representada como decimal; `0.20` equivale a 20 %. |
| Timeout | Agotamiento de espera; no demuestra éxito ni fallo del procesamiento. |
| Transición | Cambio permitido entre estados de la misma operación. |
| Vigencia | Intervalo medido en UTC con límite exclusivo. Oferta preferencial: 30 días calendario desde emisión. Oferta base: el mismo intervalo de 24 horas de la versión de preevaluación que la habilitó. Simulación exitosa: 30 minutos desde creación. |
| Versión activa de preevaluación | Última versión concluyente que no ha vencido ni ha sido reemplazada. Una respuesta `NO_DISPONIBLE` posterior no la reemplaza ni la invalida. |
| Oportunidad de crédito | Unidad de serialización de aceptación, simulación y contratación. Se identifica por `ofertaId` en el recorrido preferencial o por `solicitudId` en el regular. |
| Solicitud regular activa | Solicitud regular que todavía no ha cerrado con contrato ni sin contrato. Nace antes de solicitar su primera preevaluación y solo puede existir una por cliente; las reconsultas versionan su preevaluación sin cambiar su identidad. |
| SOLICITUD_ESPERANDO_AUTORIZACION | Estado no terminal de una solicitud cuya preevaluación exige evaluación adicional y cuya autorización todavía está `PENDIENTE`. No permite simular ni contratar. |
| SOLICITUD_FALLIDA_RECUPERABLE | Fallo técnico confirmado no terminal que permite reintentar de forma controlada la misma solicitud. |
| SOLICITUD_FALLIDA_DEFINITIVA | Fallo técnico confirmado terminal de una solicitud. |
| SIMULACION_FALLIDA_RECUPERABLE | Fallo técnico confirmado no terminal que permite recalcular de forma controlada la misma simulación. |
| SIMULACION_FALLIDA_DEFINITIVA | Fallo técnico confirmado terminal de una simulación; incluye la imposibilidad de obtener una TCEA válida. |
| Oferta reemplazada | Emisión preferencial sustituida por otra con nuevo `ofertaId`. Permanece en historial y no admite nuevas simulaciones ni contrataciones. |
| Orden de bloqueos | Orden obligatorio oportunidad–línea para toda contratación que necesite ambos recursos. Ningún flujo puede adquirirlos en orden inverso. |
| Nuevo intento de contratación | Operación con identidad propia, permitida tras verificar ausencia de contrato y de intento incierto. |
| Clave idempotente | Identifica una petición repetible. La misma clave y datos consultan el resultado original. Datos distintos producen conflicto. |
| Cohorte de visualización | Conjunto asignado por primera visualización, con seguimiento por semana de contratación. |

## Estados canónicos por propietario

- **Solicitud:** `SOLICITUD_INICIADA`, `SOLICITUD_VALIDANDO`, `SOLICITUD_ESPERANDO_AUTORIZACION`, `SOLICITUD_HABILITADA`, `SOLICITUD_RECHAZADA`, `SOLICITUD_FALLIDA_RECUPERABLE`, `SOLICITUD_FALLIDA_DEFINITIVA`, `SOLICITUD_PENDIENTE`, `SOLICITUD_CANCELADA`.
- **Simulación:** `SIMULACION_SOLICITADA`, `SIMULACION_CALCULANDO`, `SIMULACION_EXITOSA`, `SIMULACION_FALLIDA_RECUPERABLE`, `SIMULACION_FALLIDA_DEFINITIVA`, `SIMULACION_PENDIENTE`, `SIMULACION_EXPIRADA`, `SIMULACION_INVALIDADA`.
- **Contratación:** `CONTRATACION_INICIADA`, `CONTRATACION_VALIDANDO`, `CONTRATACION_REGISTRANDO`, `CONTRATACION_RECHAZADA`, `CONTRATACION_EXITOSA`, `CONTRATACION_FALLIDA_RECUPERABLE`, `CONTRATACION_FALLIDA_DEFINITIVA`, `CONTRATACION_PENDIENTE`, `CONTRATACION_EN_REVISION_MANUAL`.
- **Desembolso:** `DESEMBOLSO_SOLICITADO`, `DESEMBOLSO_PROCESANDO`, `DESEMBOLSO_CONSULTANDO_ESTADO`, `DESEMBOLSO_EN_REVISION_MANUAL`, `DESEMBOLSO_SIMULADO_EXITOSO`, `DESEMBOLSO_FALLIDO`, `DESEMBOLSO_RECHAZADO`, `DESEMBOLSO_PENDIENTE_DE_GESTION`.

Las abreviaturas históricas solo se interpretan con su propietario. API, eventos y persistencia usan siempre el nombre canónico completo. El éxito y la terminalidad son propiedades distintas (03 §8).
