# Altoque — Project Brief

**Archivo:** `01-project-brief.md`  
**Versión:** 0.1  
**Estado:** Consolidado para especificación con dependencias abiertas  
**Metodología:** BMAD / SDD  
**País de referencia:** Perú  
**Naturaleza:** Producto web de crédito de consumo para el mercado peruano

## 1. Problema y oportunidad

Altoque surge de la necesidad de solicitar un préstamo de consumo con rapidez por internet en el Perú. Las demoras durante la solicitud y la contratación pueden provocar abandono o pérdida de confianza. El efecto de reducirlas es una hipótesis de producto pendiente de validación.

La oportunidad es ofrecer un recorrido web autónomo, transparente y auditable que reduzca fricción sin sacrificar integridad financiera, unicidad contractual ni control del desembolso.

La contratación utiliza un intento durable con reserva propia. La confirmación contractual agrupa atómicamente contrato, auditoría obligatoria, orden única de desembolso y conversión de la reserva en consumo. Una respuesta perdida se recupera consultando evidencia durable del intento; nunca se infiere un cierre por ausencia de respuesta.

## 2. Usuarios y recorridos

### Público objetivo

Personas en el Perú que necesitan solicitar un crédito de consumo, incluidas quienes aún no son clientes. La aplicación admite a personas nuevas y a clientes existentes desde la primera evaluación. Poder iniciar el recorrido no garantiza una oferta ni la aprobación del crédito.

### Recorridos especificados en la primera evaluación

La visión del producto contempla que una persona del público objetivo pueda registrarse, pero el registro autónomo no forma parte de la primera evaluación. En el entorno cerrado, una persona solo participa mediante una cuenta de cliente sintético creada previamente para la evaluación y accede con credenciales sintéticas independientes de su DNI sintético. Si el cliente sintético tiene una oferta preferencial, puede simularla. Si carece de ella, sigue el flujo regular. La preevaluación versionada puede habilitar una oferta base, exigir evaluación adicional, confirmar que no existe oferta o resultar no disponible. Su resultado y autorización, si corresponde, tienen vigencia de 24 horas con límite exclusivo UTC. La oferta base vence con la versión de preevaluación que la habilitó. Una nueva versión concluyente reemplaza la activa; `NO_DISPONIBLE` no reemplaza una versión concluyente todavía vigente. Una oferta base informa monto máximo y moneda, y Altoque aplica la tasa pizarra configurada. En la primera evaluación solo se opera en PEN, con montos de S/ 500 a S/ 10 000 en incrementos de S/ 100 y plazos de 6, 12, 18 o 24 meses, sujetos además al máximo de la oferta y al saldo disponible. Los datos obligatorios faltantes no se sustituyen por valores predeterminados.

Cada emisión preferencial tiene identidad inmutable. Si existen varias ofertas preferenciales vigentes, el cliente elige una explícitamente y cada oferta constituye una oportunidad distinta sujeta a la línea compartida. En el flujo regular solo existe una solicitud activa por cliente. El cliente inicia explícitamente la solicitud regular, se crea su `solicitudId` y luego se obtiene la preevaluación vinculada a esa identidad. Una reconsulta versiona su preevaluación sin crear otra oportunidad. Una oferta o solicitud cerrada nunca se reutiliza para un nuevo préstamo.

### Límites de la primera evaluación

Los productos hipotecarios, vehiculares y empresariales quedan fuera del alcance inicial. El registro autónomo, el alta operativa de personas reales y los servicios externos de identidad, como RENIEC, quedan fuera del alcance de la primera evaluación. El contrato funcional de preevaluación se establece en RN-154–166; su contrato HTTP se documentará en OpenAPI. En el entorno cerrado se registra el contrato del flujo y se simula el desembolso.

## 3. Propuesta de valor

Altoque busca permitir que una persona solicite un crédito por la web, consulte y simule condiciones y vea el costo total antes de aceptar. La simulación presenta las condiciones del crédito antes de contratar. El cliente puede aceptar una simulación vigente y después debe seleccionar «Contratar» para iniciar la contratación. Antes del registro se comprueba que el monto solicitado no exceda el crédito disponible. Tras confirmar el registro contractual, el desembolso se inicia automáticamente.

Valor esperado: rapidez, transparencia, autonomía y confianza en que las condiciones aceptadas son las contratadas.

## 4. Modelo económico

El beneficio esperado para el banco proviene de los intereses de créditos efectivamente desembolsados. La contratación por sí sola no garantiza ingresos. No se han aprobado ticket promedio, volumen, morosidad, costos ni proyecciones.

## 5. KPIs

### KPI-01 — Conversión semanal

**Meta:** al menos 50 %.  
**Decisiones DEC-88 y DEC-99:** la primera visualización asigna la cohorte de origen. La semana de contratación asigna la conversión dentro del seguimiento de esa cohorte. Las visualizaciones repetidas de una misma oferta por el mismo cliente se deduplican.

```text
Conversión semanal (%) =
ofertas únicas contratadas atribuibles a la semana /
ofertas únicas visualizadas del conjunto definido × 100
```

La definición exacta del conjunto y el tratamiento del denominador intersemanal permanecen en PA-05; por ello la fórmula todavía no es implementable sin esa decisión.

### KPI-02 — Tiempo activo hasta aceptación

**Meta:** promedio máximo de cinco minutos.  
**Inicio:** primera visualización contabilizable.  
**Fin:** aceptación confirmada.  
**Decisión DEC-53:** sumar intervalos entre eventos válidos de interacción, con máximo de cinco minutos por intervalo.

```text
T_activo = Σ min(Δt entre eventos válidos consecutivos, 5 minutos)
```

DEC-89 fija visualización, cambios de simulación, consulta del cronograma y aceptación. El primer intervalo, los datos incompletos y el catálogo técnico se precisan en PA-06 y PA-24.

## 6. Riesgos

| ID | Riesgo | Tratamiento aprobado |
|---|---|---|
| R-01 | Desembolso duplicado | Orden durable y adquisición exclusiva. La recuperación consulta y, si corresponde, reentrega el mismo comando con idéntico identificador y datos; el core deduplica atómicamente y solo existe una ejecución lógica. |
| R-02 | Condiciones contratadas distintas | Vinculación simulación–aceptación–contrato; nueva aceptación ante cambio. |
| R-03 | Respuesta inconclusa o contradictoria del core simulado | Timeout, desconexión, respuesta inválida, estado desconocido y `NO_ENCONTRADA` mantienen el resultado pendiente. Una contradicción con un terminal local se audita y escala sin sobrescribirlo. |
| R-04 | Oferta/simulación sin vigencia | Controlar vigencias desde la emisión o creación según RN-003, RN-057 y RN-074. |
| R-05 | Fallo parcial de contrato, auditoría, orden o consumo | Intento y reserva durables previos; confirmación atómica de contrato, auditoría, orden y conversión de reserva; recuperación automática y revisión contractual sin liberar por timeout. |
| R-06 | Concurrencia de aceptaciones y contrataciones | Serialización por oportunidad, orden obligatorio oportunidad–línea, revalidación tras espera y deduplicación de aceptación por cliente, simulación y versión. |
| R-07 | Falta de comprensión financiera | Mostrar desglose completo antes de aceptar. |
| R-08 | Datos personales reales | Uso de datos sintéticos en desarrollo, pruebas y ejemplos. Uso operativo pendiente de definición. |

## 7. Resultado esperado del MVP

El recorrido de referencia de la primera evaluación cubre solicitud, simulación, aceptación, registro contractual y desembolso simulado, con cálculo backend, auditoría, prevención de duplicados y recuperación segura. La evaluación inicial se realiza en un entorno cerrado con datos sintéticos. El flujo funcional incluye una acción explícita de contratación tras la aceptación y el desembolso automático tras el registro contractual.

## 8. Hipótesis y restricciones

- Internet y uso habitual de canales digitales: hipótesis no validada.
- Reducir demoras aumenta conversión: hipótesis no validada.
- La base de evaluación contempla clientes y ofertas sintéticos en pruebas y un core simulado. La incorporación de integraciones operativas requiere confirmar el alcance.
- Java/Spring Boot, Angular, Markdown, ADR y OpenAPI: restricciones aprobadas.

## 9. Criterios de éxito funcional

- Preaprobado. Una oferta propia vigente admite simulación y contratación con aceptación vigente. Al alcanzar el límite temporal, el registro contractual se rechaza.
- Flujo regular. La oferta base respeta el máximo informado por la preevaluación y aplica la TEA pizarra configurada. El contrato es único por solicitud regular y el monto contratado debe caber en la línea compartida.
- Identidad. Una nueva emisión preferencial usa otro `ofertaId`; una nueva solicitud regular usa otro `solicitudId`. El cliente elige explícitamente entre ofertas preferenciales vigentes.

- No se expone información de otro cliente.
- No se acepta una simulación no exitosa o inválida.
- El contrato corresponde a la aceptación vigente.
- Existe como máximo un contrato por clave aprobada.
- Un timeout no produce un segundo desembolso.
- El cliente autenticado puede consultar el estado actual de sus contrataciones y desembolsos, incluso después de volver a ingresar.
- Los errores recuperables se reintentan de forma controlada; los definitivos no se reintentan y los inciertos permanecen pendientes hasta obtener evidencia concluyente.
- La revisión manual nunca crea otra operación ni modifica un resultado terminal sin evidencia.
- La aceptación conserva evidencia de que la misma versión íntegra de condiciones estuvo disponible para el cliente antes de su acción expresa.
- Las incertidumbres se mantienen explícitas.
