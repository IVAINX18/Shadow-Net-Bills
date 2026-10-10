# Requisitos de Shadow-Net Bills

Este documento define los requisitos funcionales y no funcionales de Shadow-Net Bills, aplicación para organizar y recordar obligaciones financieras personales en Colombia. Cubre el MVP manual-primero de 8 semanas y anticipa las fases V1 y V2. El lector sin contexto previo comprende qué hace el producto, qué queda fuera y cómo se verifica cada requisito.

## Convenciones

Los requisitos usan identificadores estables para trazabilidad durante todo el proyecto.

- Requisitos funcionales: formato `RF-XXX`, donde `XXX` es un número de tres dígitos con ceros a la izquierda. Ejemplo: RF-001. Cada identificador aparece una sola vez como definición y puede citarse en la matriz de trazabilidad.
- Requisitos no funcionales: formato `RNF-XXX` con la misma regla de tres dígitos. Ejemplo: RNF-001.
- Escala MoSCoW: MUST HAVE es lo obligatorio del MVP; SHOULD HAVE corresponde a V1; COULD HAVE corresponde a V2; WON'T HAVE es lo excluido por ahora. Cada requisito indica su nivel entre paréntesis.
- Fases: MVP es la versión inicial manual-primero; V1 es OCR e importación; V2 es análisis inteligente; Futuro es integraciones con correo, calendario y banca abierta.
- Nombres canónicos que se copian literalmente en diseño y código: tabla `obligations` con columna `next_due_date`; base de API `/api/v1/obligations`; las 9 categorías fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros); offsets de recordatorio `{3,1,0}` días; métodos genéricos de pago (tarjeta, transferencia, efectivo, débito automático, otro).
- Moneda: valores en pesos colombianos con formato `$20.000`. Fechas en formato `DD/MM/AAAA`. Fecha de referencia de este documento: 07/10/2026.

## Clasificación MoSCoW

La clasificación ordena el alcance por prioridad y fase. Cada nivel incluye al menos un ítem representativo y la lista completa se detalla en las secciones siguientes.

| Nivel | Significado | Ejemplos representativos |
| --- | --- | --- |
| MUST HAVE | Obligatorio en el MVP. Sin esto no se lanza. | RF-001 registro con correo y contraseña; RF-005 crear obligación; RF-016 marcar como pagada; RF-019 recordatorio 3d por correo |
| SHOULD HAVE | Importante pero posterior. Fase V1. | RF-V1-01 importar factura desde XML de la DIAN; RF-V1-02 digitalizar factura con OCR en la nube |
| COULD HAVE | Deseable si hay capacidad. Fase V2. | RF-V2-01 detectar cambios de precio entre periodos; RF-V2-02 resumen financiero con texto generado |
| WON'T HAVE | Fuera por ahora. No se diseña ni se construye. | Procesamiento de transacciones desde la aplicación; almacenamiento de claves de banca electrónica; lectura automatizada de portales de banca; presentación como entidad financiera |

Bills no mueve fondos ni ejecuta transacciones. El usuario paga por sus propios medios y solo registra el pago en la aplicación. Tampoco guarda datos de acceso a banca electrónica ni números completos de tarjeta. La posición es deliberada: reducir riesgo y validar primero si el registro manual con recordatorios resuelve el olvido de facturas.

## Requisitos funcionales del MVP

Todos los requisitos de esta sección son MUST HAVE y pertenecen al MVP. La fuente de datos es el registro manual del usuario. No hay integraciones con bancos ni procesamiento automático en esta fase.

### Autenticación y cuenta

- RF-001 (MUST HAVE): el sistema permite registrar una cuenta con correo electrónico y contraseña. Valida formato de correo, exige longitud mínima de 8 caracteres y confirma el registro con un correo de bienvenida.
- RF-002 (MUST HAVE): el sistema permite iniciar sesión con correo y contraseña y emite un token de sesión con expiración. Tras cinco intentos fallidos consecutivos aplica bloqueo temporal progresivo.
- RF-003 (MUST HAVE): el sistema permite cerrar la sesión desde cualquier pantalla y revoca el token activo de inmediato.
- RF-004 (MUST HAVE): el sistema permite recuperar la contraseña mediante un enlace temporal enviado al correo registrado, con vigencia de 60 minutos y de un solo uso.

### Obligaciones: creación, consulta, edición y eliminación

- RF-005 (MUST HAVE): el sistema permite crear una obligación con los atributos nombre, monto en COP, categoría, fecha de vencimiento, frecuencia, método genérico de pago y notas opcionales. La fecha de vencimiento alimenta la columna `next_due_date` de la tabla `obligations`.
- RF-006 (MUST HAVE): el sistema permite listar las obligaciones del usuario con filtros por estado (pendiente, pagada, vencida), por categoría y por rango de fechas, con orden por vencimiento ascendente.
- RF-007 (MUST HAVE): el sistema permite ver el detalle de una obligación con todos sus atributos, su historial de pagos y sus recordatorios programados.
- RF-008 (MUST HAVE): el sistema permite editar cualquier atributo de una obligación pendiente y recalcula `next_due_date` cuando cambian la fecha o la frecuencia.
- RF-009 (MUST HAVE): el sistema permite eliminar una obligación previa confirmación explícita en un diálogo que advierte que la acción no se puede deshacer.
- RF-010 (MUST HAVE): el sistema valida los atributos al crear y editar: monto mayor o igual a cero, fecha con formato `DD/MM/AAAA`, frecuencia dentro de one_time, weekly, monthly, quarterly o yearly, y método dentro de tarjeta, transferencia, efectivo, débito automático u otro.

### Categorías fijas

- RF-011 (MUST HAVE): el sistema ofrece exactamente las 9 categorías fijas del catálogo: Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno y Otros. Las categorías del sistema son de solo lectura.
- RF-012 (MUST HAVE): el sistema exige asignar una categoría a cada obligación y una frecuencia de recurrencia al momento de crearla. Una suscripción del MVP se registra como obligación recurrente con categoría Suscripciones y una compra a cuotas se registra como obligación con categoría Financiero y el detalle de la cuota en notas.

### Panel y calendario

- RF-013 (MUST HAVE): el sistema muestra un panel con tarjetas de resumen: total pendiente del mes, cantidad de obligaciones vencidas, próximas a vencer en 7 días y total pagado en el mes.
- RF-014 (MUST HAVE): el sistema muestra un calendario mensual con cada obligación ubicada en su fecha de vencimiento y con marca visual distinta para pendiente, vencida y pagada. La vista se consulta con el parámetro de mes y responde con los eventos del mes.

### Pagos e historial

- RF-015 (MUST HAVE): el sistema no existe sin su acción central: permite marcar una obligación como pagada con fecha de pago y método genérico usado. Esta acción es un registro manual y no ejecuta ninguna transacción externa.
- RF-016 (MUST HAVE): el sistema genera el siguiente vencimiento automáticamente al marcar como pagada una obligación recurrente, según su frecuencia, y actualiza `next_due_date` sin duplicar el registro actual.
- RF-017 (MUST HAVE): el sistema conserva el historial de pagos por obligación con fecha, monto, método y notas, y permite consultarlo en orden cronológico inverso.
- RF-018 (MUST HAVE): el sistema calcula el estado de cada obligación (pendiente, vencida, pagada) a partir de la fecha actual en zona `America/Bogota` y la fecha de pago registrada.

### Recordatorios por correo

- RF-019 (MUST HAVE): el sistema envía un recordatorio por correo 3d antes del vencimiento de cada obligación con recordatorios activados, con nombre, monto, categoría y fecha.
- RF-020 (MUST HAVE): el sistema envía un recordatorio por correo 1d antes del vencimiento en las mismas condiciones del requisito anterior.
- RF-021 (MUST HAVE): el sistema envía un recordatorio por correo el día del vencimiento 0d cuando la obligación sigue pendiente a la hora programada de la mañana.
- RF-022 (MUST HAVE): el sistema permite activar o desactivar los recordatorios por obligación y conserva los offsets `{3,1,0}` como valores por defecto configurables a nivel global.

### Resumen, configuración, exportación y eliminación de cuenta

- RF-023 (MUST HAVE): el sistema genera un resumen mensual con total por categoría, total pagado, total pendiente y comparativo simple frente al mes anterior.
- RF-024 (MUST HAVE): el sistema permite ajustar la configuración de la cuenta: zona horaria por defecto `America/Bogota`, hora de envío de recordatorios, offsets `{3,1,0}` y preferencia de formato de moneda COP.
- RF-025 (MUST HAVE): el sistema permite exportar las obligaciones y el historial de pagos a un archivo CSV descargable y permite eliminar la cuenta con borrado completo de los datos del usuario, previa confirmación con la contraseña.

## Requisitos funcionales futuros

Los requisitos futuros no pertenecen al MVP. Se etiquetan por fase para planificar sin construir antes de tiempo. Usan identificadores con prefijo de fase para distinguirlos del conteo del MVP.

### Fase V1: OCR e importación (SHOULD HAVE)

- RF-V1-01 (SHOULD HAVE, fase V1): importar factura desde el XML de la DIAN con análisis local gratuito y precarga de nombre, monto, proveedor y fecha para confirmación del usuario.
- RF-V1-02 (SHOULD HAVE, fase V1): digitalizar una factura en PDF o imagen con OCR en la nube cuando no hay XML disponible, con confirmación humana obligatoria antes de crear la obligación.
- RF-V1-03 (SHOULD HAVE, fase V1): módulo de suscripciones que agrupa las obligaciones con categoría Suscripciones y muestra costo mensual agregado y próximas renovaciones.
- RF-V1-04 (SHOULD HAVE, fase V1): módulo de compras a cuotas que divide una obligación con categoría Financiero en cuotas con su propio vencimiento y saldo restante.

### Fase V2: análisis inteligente (COULD HAVE)

- RF-V2-01 (COULD HAVE, fase V2): detectar cambios de precio entre periodos del mismo proveedor mediante comparación determinista y avisar al usuario cuando la variación supera un umbral configurable.
- RF-V2-02 (COULD HAVE, fase V2): generar un resumen financiero en lenguaje natural con totales, tendencias y sugerencias descriptivas, sin recomendar productos financieros ni prometer ahorros.

## Requisitos no funcionales

Los requisitos no funcionales aplican al MVP salvo que indiquen una fase posterior. Definen las cualidades que el sistema mantiene bajo carga y uso real.

- RNF-001 (seguridad, MUST HAVE): toda lectura y escritura de datos pasa por la API con token válido; el cliente web nunca accede directo a la base de datos y las políticas de nivel de fila aíslan a cada usuario.
- RNF-002 (seguridad, MUST HAVE): las contraseñas se almacenan con hash con sal y costo adaptativo; nunca en texto plano ni en registros de aplicación.
- RNF-003 (seguridad, MUST HAVE): todo el tráfico viaja con TLS vigente y los datos en reposo usan cifrado del proveedor; los secretos viven en variables de entorno y nunca en el código.
- RNF-004 (rendimiento, MUST HAVE): la carga inicial del panel y del calendario completa la primera vista útil en menos de 2s con conexión estándar de banda ancha en Colombia.
- RNF-005 (rendimiento, MUST HAVE): las operaciones de crear, editar y marcar como pagada responden en menos de 1s en el percentil 95 con hasta 500 obligaciones por usuario.
- RNF-006 (disponibilidad, MUST HAVE): el servicio mantiene disponibilidad mensual de 99,5 por ciento excluyendo mantenimientos programados anunciados con 48 horas de anticipación.
- RNF-007 (escalabilidad, MUST HAVE): el monolito atiende de 100 a 500 usuarios activos del MVP sin cambios de arquitectura y escala horizontalmente la capa de API sin estado compartido.
- RNF-008 (privacidad, MUST HAVE): el tratamiento de datos personales cumple la Ley 1581 de 2012: autorización previa, expresa e informada, limitación de finalidad y atención de los derechos de conocer, actualizar, rectificar, revocar y suprimir.
- RNF-009 (privacidad, MUST HAVE): cualquier transferencia internacional de datos, propia de V1 con proveedores de OCR, exige consentimiento explícito y acuerdo de tratamiento; en el MVP no hay transferencias porque todo se procesa en la infraestructura declarada.
- RNF-010 (mantenibilidad, MUST HAVE): el código sigue la estructura de monolito documentada, con convenciones de nombres y ramas, y cada cambio relevante actualiza su documento correspondiente.
- RNF-011 (accesibilidad, MUST HAVE): las pantallas cumplen WCAG 2.1 nivel AA: contraste mínimo 4.5:1 en texto, foco visible por teclado, alternativas de texto en iconos y mensajes de error asociados a sus campos.
- RNF-012 (portabilidad, MUST HAVE): el diseño es responsive con enfoque mobile-first y funciona en los navegadores vigentes Chrome, Edge, Firefox y Safari sin complementos.
- RNF-013 (confiabilidad, MUST HAVE): el programador de recordatorios envía cada aviso una sola vez mediante clave de idempotencia en el registro de notificaciones y reintenta los fallos transitorios con espera exponencial.
- RNF-014 (observabilidad, MUST HAVE): los registros de aplicación excluyen contraseñas, tokens y montos asociados a identidad; los errores exponen un código público y guardan el detalle solo en el servidor.
- RNF-015 (compatibilidad, SHOULD HAVE): la exportación CSV usa codificación UTF-8 con encabezados estables y abre correctamente en hojas de cálculo vigentes.
- RNF-016 (respaldo, MUST HAVE): la base de datos cuenta con respaldo diario automatizado con retención de 7 días y prueba de restauración antes del lanzamiento.

## Matriz de trazabilidad resumida

La matriz resume el vínculo entre requisito, caso de uso y fase. Los casos de uso UC-XX se definen en el documento de casos de uso del producto. Esta tabla usa una fila por requisito del MVP y agrupa los futuros al final.

| Requisito | Caso de uso | Fase |
| --- | --- | --- |
| RF-001 | UC-12 variante registro | MVP |
| RF-002 | UC-12 variante acceso | MVP |
| RF-003 | UC-12 variante salida | MVP |
| RF-004 | UC-12 recuperar contraseña | MVP |
| RF-005 | UC-01 registrar obligación | MVP |
| RF-006 | UC-01 variante consulta | MVP |
| RF-007 | UC-01 variante detalle | MVP |
| RF-008 | UC-02 editar obligación | MVP |
| RF-009 | UC-03 eliminar obligación | MVP |
| RF-010 | UC-01 y UC-02 validaciones | MVP |
| RF-011 | UC-01 categorías | MVP |
| RF-012 | UC-01 recurrencia y categoría | MVP |
| RF-013 | UC-09 variante panel | MVP |
| RF-014 | UC-08 consultar calendario | MVP |
| RF-015 | UC-04 marcar como pagada | MVP |
| RF-016 | UC-05 generar siguiente vencimiento | MVP |
| RF-017 | UC-04 variante historial | MVP |
| RF-018 | UC-08 y UC-09 estados | MVP |
| RF-019 | UC-06 y UC-07 recordatorio 3d | MVP |
| RF-020 | UC-06 y UC-07 recordatorio 1d | MVP |
| RF-021 | UC-06 y UC-07 recordatorio 0d | MVP |
| RF-022 | UC-06 configurar recordatorios | MVP |
| RF-023 | UC-09 ver resumen mensual | MVP |
| RF-024 | UC-06 y ajustes de cuenta | MVP |
| RF-025 | UC-10 exportar y UC-11 eliminar cuenta | MVP |
| RF-V1-01 a RF-V1-04 | UC-15 importar factura y variantes | V1 |
| RF-V2-01 a RF-V2-02 | UC-16 revisar análisis | V2 |

Los requisitos RNF-001 a RNF-016 aplican de forma transversal y se verifican en las pruebas de lanzamiento del MVP, salvo RNF-015 que acompaña a la exportación desde el primer día y se mantiene en V1 y V2.

## Supuestos y exclusiones

El MVP supone registro manual, un solo usuario por cuenta, zona horaria `America/Bogota` y correo como único canal de aviso. Quedan fuera del MVP el OCR, el módulo de suscripciones como pantalla propia, el módulo de cuotas como pantalla propia, la analítica avanzada, la detección de cambios de precio, las notificaciones push, el acceso con Google, el segundo factor de autenticación, el modo oscuro, la lectura de correo, la sincronización con calendarios externos, el procesamiento de transacciones desde la aplicación, la banca abierta y la aplicación nativa. Cada exclusión corresponde a un nivel WON'T HAVE o a una fase posterior y no se interpreta como promesa de fecha.
