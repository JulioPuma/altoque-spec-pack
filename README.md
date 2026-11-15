# Altoque — Spec Pack

Este repositorio contiene la especificación de **Altoque**, un producto web de crédito de consumo para el mercado peruano. Se elaboró con la metodología **BMAD / SDD** (Spec-Driven Development) asistida por agentes de IA.

Aquí no hay código de producción: hay documentos, prompts de agentes y prototipos visuales. El objetivo es que un agente de IA pueda usar este material como contexto para implementar el sistema (Java/Spring Boot, Angular y PostgreSQL).

La primera evaluación del producto se hará en un **entorno cerrado con datos sintéticos**. Los clientes son cuentas sintéticas precargadas y el desembolso es simulado, sin dinero real.

> ¿Es tu primera vez aquí? Empieza por [business/business-knowledge.md](business/business-knowledge.md), que explica el proyecto en lenguaje de negocio, sin tecnicismos.

---

## Resumen de resultados

### Qué se logró

- **Spec Pack de 11 documentos** (`00` a `10`): constitución, brief, alcance, 261 reglas de negocio en formato EARS, glosario, preguntas abiertas, 15 historias de usuario, arquitectura con 6 ADR, backlog de 33 tareas y especificación de UI con 16 pantallas.
- **Trazabilidad verificada**: cada regla RN-001–RN-261 tiene exactamente una historia primaria y no hay IDs huérfanos de historias, pantallas, tareas ni preguntas abiertas.
- **Modelo financiero determinista**: sistema francés, TCEA por TIR y `BigDecimal` con 34 dígitos. Los vectores normativos se recalcularon y son correctos.
- **13 prototipos HTML interactivos** (P-001 a P-013) para validar los flujos con stakeholders.

### Estado actual de la auditoría final

La última auditoría ([11-reporte-spec-finalizer.md](speck-pack/11-reporte-spec-finalizer.md), 2026-11-14) concluye que el Spec Pack **todavía no está listo para implementar**.

| Criterio | Score |
|---|:-:|
| Operatividad | 3 / 5 |
| Utilidad | 4 / 5 |
| Orden | 3 / 5 |
| **Global** | **3,3 / 5** |

Hay **9 bloqueantes** por resolver:

1. A la máquina de estados de Solicitud le faltan transiciones para el vencimiento de la preevaluación y para el cierre con contrato.
2. El estado que devuelve la API al crear un intento de contratación contradice las reglas.
3. No están definidas la política de recuperación del desembolso ni los límites de reintento.
4. No hay simulador de preevaluación ni de autorización adicional.
5. Las preguntas PA-33 (versiones del stack) y PA-34 (catálogo de mensajes) están clasificadas por debajo de su impacto real.
6. Faltan endpoints.
7. Faltan entidades en el modelo de datos.
8. El glosario tiene dos definiciones que contradicen las reglas.
9. `10-validation-log.md` está vacío y las versiones de los documentos no coinciden entre sí.

El plan de corrección está en la sección 10 del reporte. El criterio de salida es **0 bloqueantes y Operatividad ≥ 4**.

---

## Estructura del workspace

```text
altoque-spec-pack/
├── business/        Visión del negocio explicada para no técnicos
├── instruccions/    Prompts de los agentes BMAD usados en cada paso
├── speck-pack/      Spec Pack oficial (documentos normativos)
└── prototipos/      Prototipos visuales HTML de la UI
```

### `business/`

Contiene [business-knowledge.md](business/business-knowledge.md), la explicación del proyecto en lenguaje de negocio, escrita desde el rol de Product Owner para personas que no conocen el tema. Resume qué es Altoque, el problema que resuelve, a quién está dirigido, el recorrido del cliente en 6 pasos, la propuesta de valor, los beneficios y las metas, la primera evaluación, las reglas de negocio en lenguaje simple y lo que queda fuera del alcance.

**Es el mejor punto de partida para quien llega por primera vez al proyecto.**

### `instruccions/`

Guarda los prompts (en XML) de cada agente BMAD, numerados según el paso del proceso en que se usaron:

| Paso | Archivos | Agente / propósito |
|---|---|---|
| 00–05 | `00-constitucion.xml` … `05-preguntas-abiertas.xml` | Generación de la constitución, el brief, el alcance, las reglas de negocio y las preguntas abiertas |
| 06 | `06-user-stories.xml`, `06-Enriquecer criterios con Agente QA.xml` | Historias de usuario y enriquecimiento de criterios por QA |
| 07 | `07-architecture.xml` | Documento de arquitectura |
| 08 | `08-task-decomposition.xml`, `08-agente-qa-backlog.xml` | Descomposición en tareas y QA del backlog |
| 09 | `09-AGENTE UI DESIGNER.xml`, `09-qa-audit-report.md` | Especificación de UI y reporte de auditoría QA (v2.0) |
| 10 | `10-prototipo-visual-p-001.xml` … `p-013.xml` | Un prompt por cada prototipo visual |
| 11 | `11-agente-consolidador-de-validacion.xml` | Consolidación de observaciones en el validation log |
| 12 | `12-agente-spec-finalizer.xml` | Auditoría final del Spec Pack |

> Nota: `05-preguntas-abiertas copy.xml` es una copia con contenido distinto del original. Conviene revisarla y eliminarla o renombrarla.

### `speck-pack/`

Es el Spec Pack oficial. Los documentos deben leerse en orden, porque cada uno se apoya en los anteriores:

| Archivo | Contenido |
|---|---|
| [00-constitucion.md](speck-pack/00-constitucion.md) | 14 principios no negociables y 19 invariantes. Prevalece sobre cualquier otro documento. |
| [01-project-brief.md](speck-pack/01-project-brief.md) | Problema, usuarios, propuesta de valor, KPIs y riesgos. |
| [02-alcance.md](speck-pack/02-alcance.md) | Capacidades MUST/SHOULD/COULD, fuera de alcance, modelo financiero y 57 criterios de aceptación transversales. |
| [03-reglas-de-negocio.md](speck-pack/03-reglas-de-negocio.md) | Reglas RN-001–RN-261 en formato EARS y las cuatro máquinas de estados (solicitud, simulación, contratación y desembolso). |
| [04-glosario.md](speck-pack/04-glosario.md) | Lenguaje ubicuo y estados canónicos por máquina. |
| [05-preguntas-abiertas.md](speck-pack/05-preguntas-abiertas.md) | Decisiones cerradas y pendientes (PA-xx), con su trazabilidad. |
| [06-user-stories.md](speck-pack/06-user-stories.md) | HU-001–HU-015 con criterios Given/When/Then, verificación INVEST y matriz canónica RN → HU → tarea. |
| [07-architecture.md](speck-pack/07-architecture.md) | Monolito modular, modelo de datos, contratos API y ADR-001–ADR-006. |
| [08-backlog.md](speck-pack/08-backlog.md) | Tareas T-000–T-032 con prioridad, tamaño, dependencias y ruta crítica. |
| [09-ui-spec.md](speck-pack/09-ui-spec.md) | Inventario de pantallas P-001–P-016, flujos, wireframes textuales y design tokens. |
| [10-validation-log.md](speck-pack/10-validation-log.md) | Log de validación de los prototipos. **Pendiente: está vacío.** |
| [11-reporte-spec-finalizer.md](speck-pack/11-reporte-spec-finalizer.md) | Auditoría final, bloqueantes, scores y plan de corrección. |

### `prototipos/`

Contiene los prototipos HTML/CSS/JS interactivos de las pantallas **P-001 a P-013** (área del cliente y área interna de revisión), generados a partir de `09-ui-spec.md`. Cada archivo se abre directamente en el navegador. La banda superior permite forzar estados como error de red, conflicto o sesión vencida.

Las instrucciones de uso, el índice de pantallas y los datos ficticios compartidos están en [prototipos/README.md](prototipos/README.md). Las pantallas P-014 a P-016 (prioridad Could) todavía no tienen prototipo.

---

## Próximos pasos

1. Completar `10-validation-log.md` con el agente consolidador (paso 11).
2. Aplicar el plan de corrección del reporte del Spec Finalizer.
3. Unificar todos los documentos como **Spec Pack v1.0** y actualizar la lista de archivos oficiales en `00-constitucion.md`.
4. Volver a ejecutar la auditoría final (paso 12).

## Convenciones

- Documentación en español y código e identificadores en inglés (PRINCIPIO-001).
- Solo se usan datos sintéticos; no se incorporan credenciales ni datos personales reales (PRINCIPIO-013).
- Los agentes de IA editan documentos, pero no ejecutan código. Las verificaciones las realiza una persona responsable o un CI autorizado.
