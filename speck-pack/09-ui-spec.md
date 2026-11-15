# Altoque — UI Specification

**Archivo:** `09-ui-spec.md`  
**Versión del Spec Pack:** 0.2  
**Estado:** Especificación propuesta para la primera evaluación  
**Idioma de la interfaz y documentación:** Español (Perú)  
**Plataforma:** Web responsive en Angular  
**Fuentes normativas:** `00-constitucion.md`, `02-alcance.md`, `06-user-stories.md` y `07-architecture.md`

## 0. Criterios transversales

- La primera evaluación usa exclusivamente cuentas y datos sintéticos. La UI no ofrece registro, recuperación de identidad, RENIEC, movimientos de dinero reales ni productos distintos del crédito de consumo.
- Todo texto visible está en español claro. Los identificadores técnicos pueden mostrarse al usuario solo cuando faciliten soporte o trazabilidad, con etiqueta comprensible y opción de copiar.
- Todos los flujos principales son operables por teclado, con orden de foco coherente, foco visible, enlace «Saltar al contenido», etiquetas programáticas, mensajes de error asociados al campo y anuncios mediante regiones `aria-live`.
- El contraste cumple WCAG 2.2 AA. El color nunca es el único medio para comunicar estado; cada estado usa texto y, cuando aporte claridad, icono con nombre accesible.
- El frontend presenta los cálculos recibidos del backend y nunca recalcula cuota, intereses, TCEA, costo total ni cronograma.
- Las fechas visibles usan formato local comprensible (`14 nov 2026, 13:45`) y, cuando exista vencimiento, indican la zona o el carácter exacto del límite. La autoridad de vigencia sigue siendo el reloj UTC del backend.
- Las acciones de aceptación, contratación y revisión son explícitas. «Aceptar condiciones» no se presenta como sinónimo de «Contratar».
- Ante una lectura no confirmada, la UI muestra «No pudimos actualizar el estado» y la última lectura confirmada; nunca infiere éxito o fallo desde caché o por el paso del tiempo.
- Los estados terminales se muestran como solo lectura. Los reintentos visibles reutilizan la misma identidad y solo aparecen si el backend declara la operación recuperable.
- En pantallas angostas, las tablas se convierten en listas o tarjetas semánticas sin perder encabezados ni relaciones de datos.

### 0.1 Actores

| Actor | Descripción |
|---|---|
| Cliente sintético | Persona con cuenta precargada que consulta, simula, acepta, contrata y revisa únicamente sus recursos. |
| Operador de revisión contractual | Actor interno con permiso para consultar evidencia y resolver casos contractuales dentro de RN-184. |
| Operador de revisión de desembolso | Actor interno con permiso para consultar evidencia y solicitar otra consulta de la operación original. |
| Auditor | Actor interno de solo lectura; no produce transiciones. |
| Responsable humano de integración | Revisa evidencias de calidad fuera de la UI de Altoque. HU-015 se implementa como puerta CI/documental y no genera una pantalla del producto. |

## 1. Inventario de pantallas

| ID | Pantalla | Actor principal | HUs que implementa | MoSCoW |
|---|---|---|---|---|
| P-001 | Inicio de sesión | Cliente sintético | HU-001 | Must |
| P-002 | Oportunidades | Cliente sintético | HU-002, HU-003, HU-010 | Must |
| P-003 | Estado de solicitud regular | Cliente sintético | HU-003, HU-011, HU-013 | Must |
| P-004 | Configurar simulación | Cliente sintético | HU-002, HU-003, HU-004, HU-013 | Must |
| P-005 | Detalle y aceptación de simulación | Cliente sintético | HU-005, HU-010 | Must |
| P-006 | Confirmación de contratación | Cliente sintético | HU-006, HU-007 | Must |
| P-007 | Resultado de contratación | Cliente sintético | HU-006, HU-007, HU-008, HU-009, HU-012, HU-013 | Must |
| P-008 | Mis operaciones | Cliente sintético | HU-010 | Must |
| P-009 | Detalle de operación | Cliente sintético | HU-007, HU-008, HU-009, HU-010, HU-012 | Must |
| P-010 | Revisión contractual — bandeja | Operador de revisión contractual | HU-012 | Must |
| P-011 | Revisión contractual — caso | Operador de revisión contractual | HU-012 | Must |
| P-012 | Revisión de desembolso — bandeja | Operador de revisión de desembolso | HU-009, HU-014 | Must |
| P-013 | Revisión de desembolso — caso | Operador de revisión de desembolso | HU-009, HU-014 | Must |
| P-014 | Historial de simulaciones | Cliente sintético | HU-004, HU-005, HU-010 | Could |
| P-015 | Comparar simulaciones | Cliente sintético | HU-004, HU-005 | Could |
| P-016 | Ayuda financiera contextual | Cliente sintético | HU-004, HU-005 | Could |

No se define una pantalla administrativa completa: está fuera de alcance. P-010 a P-013 son superficies internas mínimas, separadas por rol y orientadas exclusivamente a revisión manual. HU-015 no aparece como pantalla porque sus criterios se verifican en CI, informes y aprobación humana antes de integrar.

## 2. User Flows

### F-01 — Autenticación y entrada

1. **P-001 Inicio de sesión:** el cliente ingresa DNI sintético y contraseña, puede mostrar u ocultar la contraseña y selecciona «Ingresar».
2. Si las credenciales son válidas, el sistema crea la sesión y dirige a **P-002 Oportunidades**.
3. Si son inválidas, inactivas, no precargadas o alcanzan el umbral aprobado, P-001 conserva el DNI, limpia la contraseña y muestra un mensaje genérico sin revelar qué dato falló.
4. Si una sesión vence durante otro flujo, el sistema vuelve a P-001. Tras autenticar, puede regresar a la ruta protegida de solo lectura; una acción de comando debe confirmarse nuevamente.

### F-02 — Oferta preferencial a simulación

1. **P-002:** el cliente revisa sus ofertas preferenciales vigentes y selecciona «Simular» en una oferta concreta.
2. **P-004:** ajusta monto válido y plazo; la UI muestra máximo de oportunidad y saldo disponible informativo recibidos del backend.
3. Selecciona «Calcular simulación».
4. Si el backend publica `SIMULACION_EXITOSA`, avanza a **P-005**. Si falla, P-004 muestra la causa y, solo para fallo recuperable, «Reintentar cálculo» sobre el mismo `simulacionId`.
5. En cualquier momento anterior al comando, «Volver» retorna a P-002 sin aceptar ni contratar.

### F-03 — Solicitud regular a simulación

1. **P-002:** el cliente selecciona explícitamente «Iniciar solicitud regular»; la UI explica que no invalida una oferta preferencial vigente.
2. **P-003:** muestra el `solicitudId` y el progreso de la preevaluación.
3. Si queda `SOLICITUD_HABILITADA`, el cliente selecciona «Simular» y avanza a **P-004**.
4. Si queda `SOLICITUD_ESPERANDO_AUTORIZACION`, selecciona «Actualizar estado» en P-003 hasta obtener autorización, denegación o vencimiento; actualizar es lectura y no crea otra solicitud.
5. Si queda `SOLICITUD_RECHAZADA`, vencida o no disponible, P-003 explica el resultado y no ofrece simulación.
6. Si el estado es recuperable, «Reintentar validación» conserva el mismo `solicitudId`.
7. Si está iniciada o habilitada y no tiene dependencias, el cliente puede elegir «Cancelar solicitud», revisar el diálogo y confirmar; una nueva solicitud posterior tendrá otra identidad.

### F-04 — Revisión, aceptación y contratación

1. **P-005:** el cliente revisa resumen, costos y cronograma de una única versión.
2. Abre el cronograma si estaba contraído; la interfaz registra la interacción permitida sin alterar la simulación.
3. Marca la casilla de conformidad y selecciona «Aceptar condiciones».
4. Al confirmarse la aceptación, **P-006** muestra nuevamente las condiciones esenciales, la diferencia entre aceptar y contratar y el aviso de revalidación de saldo/vigencias.
5. El cliente selecciona «Contratar crédito» una sola vez. El botón queda deshabilitado mientras se procesa y conserva la misma clave idempotente ante repetición técnica.
6. **P-007** muestra el resultado durable: exitoso, pendiente, revisión manual, rechazo funcional o fallo definitivo. Nunca comunica éxito si la respuesta es ambigua.

### F-05 — Consulta posterior y recuperación

1. Desde la navegación global, el cliente abre **P-008 Mis operaciones**.
2. Filtra por estado o abre una operación mediante «Ver detalle».
3. **P-009** muestra identidades, estado contractual y estado de desembolso por separado, además de la última actualización confirmada.
4. «Actualizar estado» ejecuta una lectura pura. Si falla la red, conserva visualmente la lectura previa marcada como no actualizada.
5. Si el intento está pendiente o en revisión, la pantalla explica que no debe iniciarse otra contratación. Si el desembolso está pendiente, explica que se recupera la operación original sin crear otra.

### F-06 — Revisión contractual interna

1. El operador autorizado abre **P-010** y selecciona un caso en `CONTRATACION_EN_REVISION_MANUAL`.
2. **P-011** presenta intento, reserva y evidencia durable sin iniciar transiciones por la mera consulta.
3. El operador puede agregar una observación con motivo y referencia de evidencia.
4. Solo si existe evidencia concluyente permitida, elige «Resolver caso», selecciona el resultado habilitado por el backend, revisa el resumen y confirma con versión esperada.
5. Un conflicto de versión obliga a recargar; nunca se sobrescribe el estado más reciente.

### F-07 — Revisión de desembolso interna

1. El operador autorizado abre **P-012** y elige un caso en `DESEMBOLSO_EN_REVISION_MANUAL`.
2. **P-013** muestra la evidencia y el `operacionId` original.
3. Puede registrar una observación o solicitar otra consulta del mismo `operacionId`, indicando motivo.
4. La UI no permite crear un nuevo identificador, editar el payload ni cambiar un estado terminal.
5. Un auditor puede consultar las vistas autorizadas en modo lectura, sin controles de comando.

## 3. Screen Specs

### P-001 — Inicio de sesión

**Propósito:** crear una sesión segura para una cuenta sintética precargada.  
**Actor:** cliente sintético.  
**HUs:** HU-001.

**Datos:** nombre del producto, aviso «Entorno de evaluación con datos sintéticos», DNI sintético, contraseña y enlace a ayuda de acceso definida para la evaluación.

**Acciones:** ingresar DNI; ingresar, mostrar u ocultar contraseña; enviar; saltar al contenido.

**Reglas:** sin registro autónomo; mensaje de error genérico; no revelar existencia o estado de una cuenta; no persistir credenciales en almacenamiento del navegador; respetar bloqueo temporal aprobado; contraseña independiente del DNI.

**Wireframe textual:**

```text
[Saltar al contenido]
[Encabezado simple: logotipo Altoque | insignia "Evaluación"]
<main ancho máx. 480 px>
  [H1 Inicia sesión]
  [Texto: Usa las credenciales sintéticas que te entregaron]
  [Alerta informativa: No ingreses datos personales reales]
  [Resumen de error, enfocable, si aplica]
  [Label DNI sintético]
  [Input texto, inputmode=numeric, autocomplete=off]
  [Error asociado al DNI]
  [Label Contraseña]
  [Input password, autocomplete=current-password]
  [Botón secundario Mostrar/Ocultar contraseña]
  [Error asociado a contraseña]
  [Botón primario ancho completo Ingresar]
  [Estado enviando: spinner + "Verificando acceso", aria-live]
</main>
[Pie: privacidad de evaluación | accesibilidad]
```

**Estados:** inicial; validación local; enviando; credenciales no válidas; cuenta temporalmente bloqueada con texto aprobado; servicio no disponible; sesión vencida.

### P-002 — Oportunidades

**Propósito:** mostrar oportunidades propias y permitir elegir explícitamente el recorrido preferencial o regular.  
**Actor:** cliente sintético.  
**HUs:** HU-002, HU-003, HU-010.

**Datos:** ofertas propias vigentes con `ofertaId` abreviado, monto máximo, moneda y vencimiento; solicitud regular activa si existe; resumen de operaciones pendientes; última actualización confirmada.

**Acciones:** simular una oferta; ver solicitud activa; iniciar solicitud regular; ir a Mis operaciones; actualizar lista.

**Reglas:** no mostrar recursos ajenos; ofertas vencidas no son accionables; inicio regular siempre explícito y no invalida ofertas; no crear una segunda solicitud activa; registrar primera visualización sin duplicarla.

**Wireframe textual:**

```text
[Shell cliente: logotipo | Oportunidades | Mis operaciones | menú de sesión]
<main>
  [Breadcrumb: Inicio]
  [H1 Tus oportunidades]
  [Alerta de operaciones pendientes, si existe, con enlace]
  [Sección Ofertas preferenciales]
    [Texto de ayuda sobre vigencia]
    [Grid de tarjetas]
      [Tarjeta: Oferta • monto máximo • vence fecha/hora • botón Simular]
    [Vacío: "No tienes ofertas preferenciales vigentes"]
  [Sección Solicitud regular]
    [Si no existe: explicación + botón Iniciar solicitud regular]
    [Si existe: estado textual + ID abreviado + botón Ver solicitud]
  [Bloque informativo: ambos recorridos comparten la línea disponible]
</main>
```

**Estados:** cargando con esqueletos etiquetados; contenido; lista preferencial vacía; solicitud activa; error de lectura; sesión vencida.

### P-003 — Estado de solicitud regular

**Propósito:** comunicar el resultado de preevaluación/autorización y habilitar solo acciones válidas.  
**Actor:** cliente sintético.  
**HUs:** HU-003, HU-011, HU-013.

**Datos:** `solicitudId`, estado canónico con etiqueta legible, versión de preevaluación, fecha de creación/vencimiento, máximo autorizado cuando exista, moneda, estado de autorización adicional y última lectura confirmada.

**Acciones:** actualizar estado; simular; reintentar validación controlada; solicitar cancelación; volver a oportunidades.

**Reglas:** solo `SOLICITUD_HABILITADA` vigente permite simular; autorización adicional no prolonga vigencia; no completar datos ausentes; cancelación solo en estados y ausencia de dependencias permitidos; el reintento conserva identidad.

**Wireframe textual:**

```text
[Shell cliente]
[Breadcrumb: Oportunidades / Solicitud regular]
[H1 Estado de tu solicitud]
[Panel de estado: icono + etiqueta textual + explicación]
[Definición de datos: Solicitud | creada | vence | última actualización]
[Panel condicional]
  HABILITADA: [Monto máximo] [Botón primario Simular]
  ESPERANDO_AUTORIZACION: [Explicación] [Botón Actualizar estado]
  RECHAZADA/DENEGADA: [Mensaje sin acción de simulación]
  NO_DISPONIBLE: [Mensaje] [Actualizar o reintentar si backend habilita]
  FALLIDA_RECUPERABLE: [Botón Reintentar validación]
[Acciones secundarias: Cancelar solicitud, solo si permitido | Volver]
[Diálogo de cancelación: consecuencias | Mantener | Confirmar cancelación]
```

**Estados:** validando; habilitada; esperando autorización; rechazada/denegada; no disponible; vencida; cancelable; cancelada; fallo recuperable; fallo definitivo; conflicto por actualización concurrente.

### P-004 — Configurar simulación

**Propósito:** solicitar al backend una simulación para una oportunidad habilitada.  
**Actor:** cliente sintético.  
**HUs:** HU-002, HU-003, HU-004, HU-013.

**Datos:** tipo e ID de oportunidad, máximo permitido, saldo disponible informativo, moneda PEN, monto solicitado, plazos 6/12/18/24 y errores del backend.

**Acciones:** modificar monto en pasos de S/ 100; elegir plazo; calcular; reintentar el mismo cálculo si es recuperable; volver.

**Reglas:** monto entre S/ 500 y S/ 10 000 inclusive, múltiplo de S/ 100 y no mayor al máximo/saldo; plazo permitido; datos faltantes bloquean; cálculo solo backend; doble envío bloqueado.

**Wireframe textual:**

```text
[Shell cliente + breadcrumb]
[H1 Simula tu crédito]
[Resumen de oportunidad: tipo | ID abreviado | máximo | vigencia]
[Formulario]
  [Label Monto en soles]
  [Input numérico con prefijo S/ y ayuda "Incrementos de S/ 100"]
  [Controles -100 / +100 accesibles, opcionales]
  [Error de rango asociado]
  [Fieldset Plazo]
    [Radio 6 meses] [12] [18] [24]
  [Aviso: el resultado y costos serán calculados por el sistema]
  [Botón primario Calcular simulación]
  [Botón/enlace Volver]
[Alerta de error del backend; botón Reintentar solo si recuperable]
```

**Estados:** editable; error local; calculando; exitosa con redirección; parámetros rechazados; datos obligatorios ausentes; oportunidad/saldo vencido o cambiado; fallo recuperable; fallo definitivo.

### P-005 — Detalle y aceptación de simulación

**Propósito:** presentar una versión íntegra y permitir su aceptación expresa mientras siga vigente.  
**Actor:** cliente sintético.  
**HUs:** HU-005, HU-010.

**Datos:** `simulacionId`, versión/hash abreviado, creación/vencimiento, monto, plazo, cuota financiera, seguro por cuota, pago total por cuota, intereses totales, TCEA con dos decimales, costo total y cronograma completo.

**Acciones:** expandir cronograma; aceptar condiciones; volver a simular; imprimir mediante función del navegador si se desea.

**Reglas:** todos los campos obligatorios pertenecen a la misma versión; información incompleta deshabilita aceptación; vencimiento exclusivo a 30 minutos; aceptar no contrata; no usar un contador local como autoridad.

**Wireframe textual:**

```text
[Shell cliente + breadcrumb]
[H1 Revisa tu simulación]
[Banner de vigencia: "Disponible para aceptar hasta …" + estado textual]
[Resumen destacado: Monto | Plazo | Pago total mensual]
[Grid Costos]
  [Cuota financiera] [Seguro por cuota] [Intereses totales]
  [TCEA] [Costo total]
[Acordeón Cronograma de pagos, expandible por teclado]
  [Tabla: N° | fecha | saldo inicial | capital | interés | seguro | pago | saldo final]
  [En móvil: una tarjeta por cuota con los mismos rótulos]
[Nota: valores calculados por Altoque; moneda PEN]
[Checkbox: "He revisado y acepto estas condiciones de la versión …"]
[Botón primario Aceptar condiciones]
[Botón secundario Cambiar monto o plazo]
[Alerta bloqueante si falta información o venció]
```

**Estados:** vigente completa; cronograma contraído/expandido; aceptando; aceptada; repetición idempotente; vencida; invalidada; información incompleta; conflicto por contratación concurrente.

### P-006 — Confirmación de contratación

**Propósito:** separar inequívocamente aceptación y comando de contratación.  
**Actor:** cliente sintético.  
**HUs:** HU-006, HU-007.

**Datos:** ID de aceptación, versión aceptada, monto, plazo, pago mensual, TCEA, costo total, vigencias y aviso de revalidación de línea.

**Acciones:** contratar; volver al detalle sin crear intento.

**Reglas:** solo aceptación vigente; contratación crea intento/reserva y puede quedar pendiente; se preserva una clave idempotente para el mismo envío; una clave nueva no se usa para evadir guardas.

**Wireframe textual:**

```text
[Shell cliente + breadcrumb]
[H1 Confirma la contratación]
[Alerta: "Aceptar condiciones no contrató el crédito"]
[Tarjeta resumen inmutable]
  [Monto | plazo | pago total mensual | TCEA | costo total]
  [Versión aceptada | aceptación registrada]
[Lista "Al contratar"]
  [Se revalidarán vigencias y saldo]
  [Se reservará el monto]
  [El desembolso de esta evaluación es simulado]
[Botón primario Contratar crédito]
[En proceso: botón deshabilitado + "Estamos registrando tu solicitud"]
[Botón secundario Volver al detalle]
```

**Estados:** listo; procesando; conflicto de saldo/vigencia; intento existente; resultado ambiguo con paso inmediato a P-007; sesión vencida antes del comando.

### P-007 — Resultado de contratación

**Propósito:** mostrar el resultado confirmado o pendiente del mismo intento sin inducir duplicados.  
**Actor:** cliente sintético.  
**HUs:** HU-006, HU-007, HU-008, HU-009, HU-012, HU-013.

**Datos:** `intentoId`, estado contractual, reserva, contrato si existe, `operacionId` y estado de desembolso si existen, última lectura confirmada y explicación contextual.

**Acciones:** actualizar estado mediante lectura; ver detalle de operación; ir a Mis operaciones; reintentar solo si el backend habilita el mismo intento.

**Reglas:** no inferir éxito; pendiente conserva identidad/reserva; a los 15 minutos la recuperación contractual puede pasar a revisión manual, pero el tiempo no acredita resultado; desembolso separado del contrato; no ofrecer «Contratar otra vez» para la oportunidad.

**Wireframe textual:**

```text
[Shell cliente]
[H1 Resultado de tu contratación]
[Panel dominante de estado]
  EXITOSA: icono + "Contrato registrado" + ID de contrato
  PENDIENTE: icono + "Resultado pendiente" + explicación
  REVISION: icono + "En revisión manual" + objetivo operativo, sin promesa de resultado
  RECHAZADA/FALLIDA: texto y motivo funcional permitido
[Línea de progreso semántica, no basada solo en color]
  Aceptación — Contratación — Desembolso simulado
[Definición de datos: intento | reserva | contrato | operación | última lectura]
[Alerta: no inicies otra contratación mientras este intento esté pendiente]
[Botones: Actualizar estado | Ver detalle | Mis operaciones]
```

**Estados:** contratación exitosa con desembolso solicitado/procesando/exitoso; contratación pendiente; revisión contractual; fallo recuperable; fallo definitivo; rechazo; lectura no disponible.

### P-008 — Mis operaciones

**Propósito:** listar operaciones propias pendientes y resueltas después de reingresar.  
**Actor:** cliente sintético.  
**HUs:** HU-010.

**Datos:** intento, contrato, operación, estado contractual, estado de desembolso, fecha de actualización y paginación por cursor.

**Acciones:** filtrar; ver detalle; cargar página siguiente; actualizar.

**Reglas:** solo recursos propios; lista vacía válida; límite de página dentro de 1–50; lectura pura; no presentar datos de caché como actuales.

**Wireframe textual:**

```text
[Shell cliente: Oportunidades | Mis operaciones activo]
[H1 Mis operaciones]
[Filtros: Todas | Pendientes | En revisión | Resueltas]
[Texto de última actualización + botón Actualizar]
[Tabla escritorio]
  [Fecha | Intento | Contratación | Desembolso | Acción Ver detalle]
[Lista de tarjetas móvil con los mismos campos]
[Botón Cargar más si nextCursor existe]
[Vacío: "Aún no tienes operaciones" + enlace a Oportunidades]
[Error: lectura no disponible + Reintentar]
```

**Estados:** cargando; con resultados; vacío; filtrado sin resultados; error de red; página adicional; sesión vencida.

### P-009 — Detalle de operación

**Propósito:** explicar de forma separada el estado contractual y el desembolso simulado.  
**Actor:** cliente sintético.  
**HUs:** HU-007, HU-008, HU-009, HU-010, HU-012.

**Datos:** IDs persistentes, estados y versiones, monto/moneda, reserva, hitos confirmados, última lectura y mensajes de revisión.

**Acciones:** actualizar estado; copiar identificadores; volver a lista.

**Reglas:** consulta no transiciona; estados terminales inmutables; timeout/`NOT_FOUND` no son fallo; pendiente/revisión no habilita otra operación; mínima revelación para un ID ajeno.

**Wireframe textual:**

```text
[Shell cliente + breadcrumb Mis operaciones / Detalle]
[H1 Detalle de operación]
[Resumen: monto PEN | fecha | estado general textual]
[Sección Contratación]
  [Intento ID + copiar | estado | reserva | contrato ID si existe]
[Sección Desembolso simulado]
  [Operación ID + copiar | estado independiente | explicación]
[Timeline de hitos confirmados con fecha/hora]
[Alerta contextual para pendiente o revisión]
[Última lectura confirmada]
[Botón Actualizar estado] [Volver a Mis operaciones]
```

**Estados:** pendiente; revisión contractual; contratación exitosa con desembolso en cada estado; terminal exitoso/fallido; contradicción escalada sin sobrescribir terminal; lectura fallida.

### P-010 — Revisión contractual: bandeja

**Propósito:** listar exclusivamente casos contractuales autorizados para revisión.  
**Actor:** operador de revisión contractual; auditor en modo lectura.  
**HUs:** HU-012.

**Datos:** `intentoId`, antigüedad, estado, monto de reserva, última evidencia/actualización, versión y paginación.

**Acciones:** filtrar; abrir caso; actualizar bandeja; cargar más.

**Reglas:** rol contractual; solo estado de revisión correspondiente; mínima revelación; abrir un caso no produce transición.

**Wireframe textual:**

```text
[Shell interno: identidad/rol | Revisión contractual | Revisión desembolso según permisos]
[H1 Revisión contractual]
[Filtros por antigüedad y estado permitido]
[Tabla: Intento | En revisión desde | Reserva | Última evidencia | Versión | Ver caso]
[Paginación por cursor]
[Vacío: "No hay casos contractuales asignados"]
[403: pantalla de acceso denegado sin detalles del recurso]
```

**Estados:** cargando; lista; vacío; acceso denegado; error; versión/lista actualizada.

### P-011 — Revisión contractual: caso

**Propósito:** consultar evidencia, registrar observaciones y resolver solo con evidencia concluyente.  
**Actor:** operador de revisión contractual; auditor en modo lectura.  
**HUs:** HU-012.

**Datos:** intento, estado/versión, reserva, oportunidad, aceptación, evidencia durable, correlación, observaciones y auditoría visible autorizada.

**Acciones:** registrar observación; resolver caso entre opciones habilitadas por servidor; recargar ante conflicto; copiar correlación.

**Reglas:** comandos con motivo no vacío, correlación, versión esperada e idempotencia; reserva se conserva sin evidencia; no crear otro intento ni liberar reserva por inferencia; auditor sin comandos.

**Wireframe textual:**

```text
[Shell interno + breadcrumb]
[H1 Caso contractual]
[Banner: estado | tiempo en revisión | versión]
[Resumen: intento | oportunidad | reserva | aceptación]
[Sección Evidencia durable]
  [Lista cronológica: fuente | instante UTC | resultado | correlación]
[Sección Observaciones]
  [Historial]
  [Textarea Motivo/observación + referencia de evidencia + Guardar]
[Panel Resolver caso, oculto para auditor]
  [Opciones devueltas por backend]
  [Textarea Motivo obligatorio]
  [Resumen de impacto]
  [Botón Resolver caso + diálogo de confirmación]
[Conflicto 409: alerta + botón Recargar; no reenviar automáticamente]
```

**Estados:** lectura; editando observación; enviando; evidencia insuficiente; resolución habilitada; conflicto de versión; resuelto; acceso denegado.

### P-012 — Revisión de desembolso: bandeja

**Propósito:** listar casos de desembolso pendientes de revisión manual.  
**Actor:** operador de revisión de desembolso; auditor en modo lectura.  
**HUs:** HU-009, HU-014.

**Datos:** orden/`operacionId`, contrato, estado, antigüedad, última consulta/evidencia, versión y paginación.

**Acciones:** filtrar; abrir; actualizar; cargar más.

**Reglas:** solo rol correspondiente; consultar no invoca el core; no mostrar controles contractuales.

**Wireframe textual:**

```text
[Shell interno]
[H1 Revisión de desembolsos]
[Filtros permitidos]
[Tabla: Operación | Contrato | En revisión desde | Última evidencia | Versión | Ver caso]
[Paginación por cursor]
[Vacío / error / acceso denegado]
```

**Estados:** cargando; lista; vacío; acceso denegado; error.

### P-013 — Revisión de desembolso: caso

**Propósito:** inspeccionar evidencia y solicitar otra consulta de la misma operación sin duplicarla.  
**Actor:** operador de revisión de desembolso; auditor en modo lectura.  
**HUs:** HU-009, HU-014.

**Datos:** orden, `operacionId`, contrato, payload original en modo lectura, estado/versión, evidencia, observaciones, correlación.

**Acciones:** registrar observación; solicitar otra consulta de la operación original; recargar.

**Reglas:** mismo `operacionId`; payload no editable; motivo/correlación/versión/idempotencia obligatorios; sin creación de operación ni cambio de terminal; auditor solo lectura.

**Wireframe textual:**

```text
[Shell interno + breadcrumb]
[H1 Caso de desembolso]
[Banner: estado | versión | última evidencia]
[Resumen inmutable: operación | contrato | monto | moneda]
[Payload original colapsable, solo lectura]
[Timeline de evidencia]
[Observaciones: historial + formulario con motivo y referencia]
[Panel Acción, oculto para auditor]
  [Texto: "Se consultará nuevamente la operación original …"]
  [Motivo obligatorio]
  [Botón Solicitar otra consulta]
  [Diálogo confirma mismo operationId; no permite editarlo]
[Alerta de conflicto/terminal: recargar, sin reenvío automático]
```

**Estados:** lectura; consulta permitida; consulta en curso; evidencia aún inconclusa; terminal de solo lectura; conflicto; acceso denegado.

## 4. Design Tokens

Los valores son una propuesta visual, no parámetros de negocio. Deben validarse con herramientas de contraste y pruebas en dispositivos objetivo antes de integrar.

### 4.1 Color

| Token | Valor sugerido | Uso |
|---|---|---|
| `color-primary-700` | `#174EA6` | Botón primario, enlaces y foco sobre fondo claro |
| `color-primary-800` | `#123B7A` | Hover/pressed primario |
| `color-secondary-700` | `#0B6B5B` | Elementos secundarios y estados informativos positivos |
| `color-accent-500` | `#F5A623` | Acento limitado; nunca texto pequeño sobre blanco |
| `color-bg-page` | `#F7F9FC` | Fondo general |
| `color-bg-surface` | `#FFFFFF` | Tarjetas, diálogos, formularios |
| `color-text-primary` | `#172033` | Texto principal |
| `color-text-secondary` | `#4B5565` | Texto auxiliar |
| `color-border` | `#C9D2E3` | Bordes y divisores |
| `color-success-700` | `#18794E` | Éxito, acompañado de texto/icono |
| `color-warning-800` | `#8A4B08` | Advertencia y pendientes |
| `color-danger-700` | `#B42318` | Error y acciones destructivas |
| `color-info-700` | `#175CD3` | Información y revisión |
| `color-focus` | `#7C3AED` | Anillo de foco de alto contraste |

### 4.2 Tipografía

- Familia: `Inter, "Segoe UI", Roboto, Arial, sans-serif`; usar fuente del sistema como fallback y no bloquear la interfaz por descarga.
- Base: 16 px; altura de línea 1.5.
- H1: 32/40 px, peso 700; móvil 28/36 px.
- H2: 24/32 px, peso 700.
- H3: 20/28 px, peso 600.
- Cuerpo: 16/24 px, peso 400.
- Texto auxiliar: 14/20 px; no usar menos de 14 px para información funcional.
- Valores monetarios: cifras tabulares (`font-variant-numeric: tabular-nums`) sin reducir la legibilidad.

### 4.3 Espaciado, forma y elevación

- Escala de espaciado: 4, 8, 12, 16, 24, 32, 48 y 64 px.
- Ancho máximo de contenido cliente: 1200 px; formularios: 640 px; lectura financiera: 960 px.
- Radio: controles 8 px, tarjetas 12 px, diálogos 16 px.
- Borde estándar: 1 px; borde enfatizado: 2 px.
- Sombra de tarjeta: `0 1px 3px rgba(23,32,51,.12)`; no depender de sombra para delimitar superficies.

### 4.4 Componentes y objetivos táctiles

| Componente | Tamaño mínimo | Reglas |
|---|---:|---|
| Botón | 44 px de alto | Etiqueta verbal; loading preserva ancho; disabled no sustituye explicación. |
| Input/select | 48 px de alto | Label visible; ayuda y error asociados mediante `aria-describedby`. |
| Checkbox/radio | Área activa 44 × 44 px | Control nativo o semántica equivalente; label completo clicable. |
| Icon button | 44 × 44 px | Nombre accesible y tooltip que no sea la única etiqueta. |
| Tarjeta accionable | Variable | Una acción primaria inequívoca; foco visible; no anidar controles interactivos inválidos. |
| Diálogo | 320–640 px | Foco inicial razonable, trampa de foco y retorno al disparador; Escape salvo comando irreversible en proceso. |
| Tabla | Fila mínima 48 px | Encabezados `th`; caption; alternativa responsive conservando rótulos. |

### 4.5 Breakpoints y comportamiento responsive

- `sm`: 0–599 px, una columna y navegación compacta.
- `md`: 600–959 px, una o dos columnas según contenido.
- `lg`: 960–1279 px, grillas y tablas completas.
- `xl`: 1280 px o más, contenido centrado con ancho máximo.
- No existe desplazamiento horizontal de página a 320 CSS px. El cronograma puede usar contenedor de tabla con aviso accesible o tarjetas, prefiriendo tarjetas en móvil.

### 4.6 Estados y mensajes

- **Cargando:** skeleton o spinner con texto accesible; no desplazar el foco sin causa.
- **Vacío:** explicar qué significa y ofrecer una acción válida, si existe.
- **Error de campo:** resumen enfocable y mensaje junto al campo.
- **Error de red:** no borrar datos confirmados; marcar la fecha de última lectura.
- **Pendiente:** icono + texto + siguiente paso; sin animación que sugiera progreso garantizado.
- **Éxito:** solo tras evidencia durable del backend.
- **Acción destructiva:** botón danger, consecuencias concretas y confirmación; P-003 cancelar solicitud es el único caso del recorrido cliente definido aquí.

## 5. Navigation Map

```text
Altoque
├── P-001 Inicio de sesión
└── Área autenticada del cliente
    ├── P-002 Oportunidades (inicio)
    │   ├── Oferta preferencial seleccionada
    │   │   └── P-004 Configurar simulación
    │   │       └── P-005 Detalle y aceptación
    │   │           └── P-006 Confirmación de contratación
    │   │               └── P-007 Resultado de contratación
    │   └── Solicitud regular
    │       └── P-003 Estado de solicitud
    │           └── P-004 Configurar simulación
    │               └── P-005 Detalle y aceptación
    │                   └── P-006 Confirmación de contratación
    │                       └── P-007 Resultado de contratación
    ├── P-008 Mis operaciones
    │   └── P-009 Detalle de operación
    ├── P-014 Historial de simulaciones [Could]
    │   └── P-005 Detalle y aceptación (si sigue vigente)
    ├── P-015 Comparar simulaciones [Could]
    └── P-016 Ayuda financiera contextual [Could]

Área interna (sesión y autorización separadas)
├── P-010 Revisión contractual — bandeja
│   └── P-011 Revisión contractual — caso
└── P-012 Revisión de desembolso — bandeja
    └── P-013 Revisión de desembolso — caso
```

### 5.1 Navegación global

- Cliente: «Oportunidades», «Mis operaciones» y menú de sesión. Las pantallas Could solo aparecen cuando se implementen y no desplazan las rutas Must.
- Operador: solo módulos concedidos por rol. Un operador contractual no recibe acceso implícito a revisión de desembolsos ni viceversa.
- Auditor: rutas internas autorizadas en modo lectura, sin controles que produzcan transiciones.
- Las rutas profundas exigen sesión válida y pertenencia/autorización comprobada en backend. La UI no usa ocultamiento de controles como única medida de seguridad.

## 6. Trazabilidad y límites de implementación

| Decisión de UI | Fuente |
|---|---|
| Separar aceptar y contratar | HU-005, HU-006; PRINCIPIO-004 y PRINCIPIO-006 |
| Mostrar condiciones completas y cronograma de una versión | HU-005; PRINCIPIO-003 a PRINCIPIO-005 |
| Estados contractual y de desembolso independientes | HU-007 a HU-010; PRINCIPIO-009 |
| Mantener «pendiente» ante ambigüedad | HU-009, HU-012; PRINCIPIO-008 |
| Revisión interna con permisos mínimos | HU-012, HU-014; arquitectura §4.11 |
| Teclado, etiquetas, foco y contraste | PRINCIPIO-012; arquitectura §6.3 |
| Español visible y código en inglés | PRINCIPIO-001 |
| Datos sintéticos y ausencia de registro | PRINCIPIO-013; M-01 y fuera de alcance |

Quedan deliberadamente fuera de esta especificación: fórmulas de KPI no aprobadas, parámetros concretos de contraseña/sesión/rate limit aún reservados a la especificación de seguridad, un panel administrativo bancario completo, notificaciones externas y cualquier acción que ejecute dinero real. Su ausencia no debe suplirse con supuestos de interfaz.
