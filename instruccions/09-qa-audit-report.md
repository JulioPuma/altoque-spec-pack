# Reporte de auditoría QA — Spec Pack Altoque

**Fecha de auditoría:** 2026-11-14  
**Versión del reporte:** 2.0  
**Documentos revisados:** `00-constitucion.md` a `08-backlog.md`  
**Estado:** Reporte QA vigente; reemplaza la auditoría anterior

## Resumen ejecutivo

El Spec Pack mejoró sustancialmente su trazabilidad: las 15 HUs Must tienen arquitectura y tareas, RN-001–RN-261 cuentan con matriz canónica, y todas las tareas funcionales o habilitadoras poseen una referencia explícita. No obstante, la auditoría nueva concluye que todavía existen hallazgos mayores por resolver antes de declarar el pack listo para implementación.

| Dimensión | Resultado actual |
|---|---|
| Reglas Must → HUs | Completa mediante la matriz canónica de `06-user-stories.md`. |
| HUs Must → arquitectura | Completa para HU-001–HU-015. |
| Arquitectura → backlog | Completa en términos generales; algunos artefactos pendientes aún carecen de decisión cerrada. |
| Backward traceability | Completa: las tareas funcionales trazan a HUs y los Enablers a principios, ADR o RN. |
| Consistencia constitucional | Parcial: el alcance permite ambos recorridos, pero la narrativa de Constitución y brief conserva una formulación ambigua. |
| Preguntas abiertas bloqueantes | PA-13 y PA-24 quedaron cerradas; no permanecen críticos abiertos de esta auditoría. |
| Preparación para implementar | No alcanzada. Se requieren decisiones humanas y correcciones documentales. |

Hallazgos abiertos:

- 0 críticos abiertos y 2 resueltos.
- 7 mayores.
- 3 sugerencias.

---

# Hallazgos críticos

## ✅ C2-01 — Resuelto: política completa de cancelación de solicitud

La máquina de solicitud conserva:

```text
SOLICITUD_INICIADA o SOLICITUD_HABILITADA
  → Cancelación permitida
  → SOLICITUD_CANCELADA
```

PA-13 fue cerrada. RN-260 establece que solo el cliente propietario puede cancelar idempotentemente desde `SOLICITUD_INICIADA` o `SOLICITUD_HABILITADA` y únicamente cuando no existe ninguna simulación, intento, reserva ni contrato.

La cancelación se serializa por oportunidad, revalida guardas, se audita, no reutiliza la identidad y una solicitud posterior recibe otro `solicitudId` y nueva preevaluación.

**Estado:** resuelto en alcance, RN-260, glosario, PA-13, HU-003, arquitectura, T-007, T-031 y matriz canónica.

## ✅ C2-02 — Resuelto: catálogo mínimo de eventos KPI aprobado

PA-24 quedó cerrada y RN-261 define los eventos `OPPORTUNITY_VIEWED`, `SIMULATION_PARAMETERS_CHANGED`, `SCHEDULE_VIEWED`, `ACCEPTANCE_CONFIRMED` y `CONTRACT_CONFIRMED`.

El catálogo fija identidad, campos obligatorios, instante UTC, correlación, fuente, versión, esquema, deduplicación, tratamiento append-only y eventos tardíos. La primera visualización se deduplica por cliente y oportunidad; las interacciones posteriores pueden conservarse separadamente para KPI-02.

**Estado:** resuelto en RN-261, PA-24, HU-010, modelo `InteractionEvent`, API de captura, T-003, T-004, T-020, T-032 y matriz canónica. PA-05 y PA-06 continúan abiertas únicamente para cálculo y agregación.

---

# Hallazgos mayores

## 🟡 M2-01 — La narrativa del flujo regular permanece ambigua en Constitución y brief

El alcance, RN-172, HU-003, arquitectura y T-007 permiten expresamente que cualquier cliente sintético inicie el flujo regular, tenga o no oferta preferencial. La Constitución delega la delimitación de la primera evaluación al alcance, por lo que no existe una prohibición constitucional explícita. Sin embargo, su introducción y `01-project-brief.md` todavía describen “con oferta → preferencial” y “sin oferta → regular”, lo que puede leerse como recorridos exclusivos.

**Acción recomendada:** aclarar la narrativa de `00-constitucion.md`, `01-project-brief.md` y PA-38 durante el siguiente cambio de versión, indicando que la selección explícita del flujo regular también está disponible con oferta vigente y nunca se inicia automáticamente.

## 🟡 M2-02 — PA-32 mantiene incompleta la política transversal de auditoría

PA-32 continúa abierta para creación de registros, actores automáticos, observaciones sin cambio de estado, atomicidad fuera del contrato y distinción entre versión de entidad y versión financiera. RN-081, RN-082, T-020, T-014 y T-025 dependen de estas decisiones.

**Acción recomendada:** cerrar PA-32 con una tabla por tipo de evento auditable que defina actor, atomicidad, versión, campos obligatorios y tratamiento de observaciones sin transición.

## 🟡 M2-03 — PA-34 deja incompleto el contrato de consulta y revisión

HU-010 y la arquitectura fijan lecturas básicas, pero PA-34 mantiene abiertos los detalles visibles, datos disponibles durante revisión y catálogo de motivos/mensajes. T-004 debe publicar el OpenAPI completo y T-018/T-025 implementar esas respuestas.

**Acción recomendada:** aprobar una política mínima de representación por rol y estado antes de T-004, incluyendo campos permitidos, redacción de mínima revelación y códigos canónicos de motivo.

## 🟡 M2-04 — PA-31 deja sin política operable los ajustes de consumo

RN-033 y RN-104 permiten un ajuste auditable para corregir un terminal o liberar consumo después de un fallo definitivo, pero PA-31 no define causas, actor autorizado, evidencia, identidad, deduplicación ni límite del efecto.

**Acción recomendada:** definir la política y crear una HU/tarea Must para el ajuste, o declarar expresamente que ningún ajuste de consumo se ejecutará en la primera evaluación y conservar RN-104 como condición futura no implementada.

## 🟡 M2-05 — Las tareas de cierre exceden probablemente el límite de dos horas

El backlog afirma que cada tarea puede completarse por un agente en menos de dos horas. Ese límite no es creíble para:

- T-026: cobertura, SAST, ASVS, accesibilidad y pruebas arquitectónicas.
- T-030: política completa de seguridad y threat model con aprobación humana.
- T-031: pruebas exhaustivas de cuatro máquinas de estados.

**Acción recomendada:** dividir cada una en subtareas independientes de producción de artefactos, automatización, revisión humana y aprobación. Mantener una tarea agregadora de cierre sin estimación de implementación.

## 🟡 M2-06 — El backlog mezcla decisiones ya completadas con trabajo pendiente

T-000 registra una aprobación que ya ocurrió el 2026-11-14, pero el backlog no posee campo de estado y la ruta crítica la presenta como trabajo futuro. Esto distorsiona avance y dependencias.

**Acción recomendada:** añadir `Estado` a todas las tareas o mover T-000 a un registro de decisiones completadas. Si se conserva, marcarla `Done` y excluirla del trabajo restante y de las estimaciones.

## 🟡 M2-07 — La versión documental no refleja los cambios normativos

La Constitución y los documentos oficiales 01–05 continúan en versión 0.1; `06-user-stories.md` declara versión 0.2. Desde la base 0.1 se añadieron HU-011–HU-015, RN-259, nuevos roles, ADR aprobados y una decisión diferente sobre acceso al flujo regular.

**Acción recomendada:** establecer una única versión del Spec Pack, registrar changelog y actualizar metadatos de todos los documentos afectados después de aprobar la enmienda constitucional. No presentar como misma versión dos baselines normativas distintas.

---

# Sugerencias

## 🟢 S2-01 — Ordenar físicamente el backlog por dependencias e identificadores

T-030–T-032 aparecen antes de T-026 para respetar dependencias, pero rompen la secuencia numérica visual.

**Acción recomendada:** renumerar de forma controlada o separar secciones `Enablers`, `Implementación`, `Verificación` y `Puertas de cierre`, conservando un mapa de IDs anteriores.

## 🟢 S2-02 — Añadir estado y responsable a cada tarea

**Acción recomendada:** incorporar `Estado`, `Responsable de aprobación` y `Evidencia esperada` para distinguir especificación, implementación, ejecución humana y aprobación.

## 🟢 S2-03 — Normalizar el nombre de la carpeta

La carpeta sigue llamándose `speck-pack`, mientras los documentos usan “Spec Pack”.

**Acción recomendada:** renombrarla a `spec-pack` en una operación controlada cuando no rompa enlaces ni automatizaciones.

---

# Matriz resumida de trazabilidad

| Capacidad | Reglas/HUs | Arquitectura | Backlog | Resultado QA |
|---|---|---|---|---|
| Acceso y pertenencia | RN-001–004, RN-065, RN-121–124, RN-237–243; HU-001–HU-003 | Identity & Access; Opportunity & Application | T-004–T-007, T-021, T-030 | Cubierta; narrativa de recorridos aún ambigua en brief/Constitución |
| Simulación financiera | RN-005–014, RN-036–041, RN-060–064, RN-073–077, RN-083–084, RN-186–201; HU-004–HU-005 | Financial Simulation; Web Client | T-008–T-011, T-019, T-029 | Completa documentalmente |
| Contratación y recuperación | RN-015–018, RN-029–035, RN-043–050, RN-067–071, RN-078–099, RN-107–143, RN-178–185, RN-223–230, RN-248–259; HU-006, HU-007, HU-012, HU-013 | Acceptance & Contracting; Identity & Access; Audit | T-012–T-014, T-020–T-024, T-030–T-031 | Parcial: PA-32 y política de ajustes PA-31 |
| Desembolso y revisión | RN-019–025, RN-042, RN-051–055, RN-100–104, RN-129–153, RN-231–240; HU-008, HU-009, HU-014 | Disbursement & Recovery; Audit | T-015–T-017, T-022, T-025, T-028, T-030–T-031 | Completa salvo política transversal de ajustes/auditoría |
| Estados | RN-069–070, RN-081–082, RN-175–185, RN-217–236, RN-260; HU-003–HU-009, HU-012–HU-014 | Cuatro máquinas independientes | T-007, T-024, T-031 | Completa documentalmente; cancelación definida por RN-260 |
| KPI y observabilidad | RN-026–027, RN-261; HU-010 | Audit & Observability | T-003, T-004, T-020, T-032 | Completa para captura de evidencia; fórmulas PA-05/PA-06 continúan fuera de implementación |
| Calidad y seguridad | RN-028, RN-080, RN-244–247; HU-015 | CI, seguridad y accesibilidad | T-026, T-030–T-031 | Planificada; falta producir y aprobar artefactos |

## Dictamen

El Spec Pack no tiene tareas huérfanas, reglas Must sin HU primaria ni críticos abiertos en esta auditoría, pero todavía no cumple la condición de salida para implementación debido a los hallazgos mayores.

Condiciones mínimas pendientes:

1. Aclarar la narrativa del flujo regular y versionar el baseline.
2. Cerrar o acotar PA-31, PA-32 y PA-34.
3. Dividir T-026, T-030 y T-031 en unidades verificables menores a dos horas.
4. Distinguir decisiones completadas de trabajo pendiente en el backlog.

Después de estas correcciones debe ejecutarse una auditoría de cierre de trazabilidad y consistencia. Las pruebas y evidencias de ejecución siguen correspondiendo a una persona responsable o CI autorizado.
