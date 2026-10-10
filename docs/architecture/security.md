# Seguridad de Shadow-Net Bills: modelo de amenazas y controles

Este documento define los principios, el modelo de amenazas, los controles técnicos, la auditoría, la hoja de ruta de autenticación avanzada y el plan de respuesta a incidentes de Shadow-Net Bills. Aplica al MVP manual-primero de 8 semanas descrito en la especificación: aplicación para organizar y recordar obligaciones financieras personales en Colombia, sin mover dinero, sin almacenar credenciales bancarias ni números completos de tarjeta, y sin integraciones bancarias en el MVP. La IA y el OCR pertenecen a V1 y posterior; este documento marca cada amenaza con su fase para no insinuar que el MVP expone superficie de subidas u OCR.

Audiencia: equipo de desarrollo, responsables de producto y revisores técnicos. No constituye asesoramiento jurídico; el marco regulatorio se detalla en ../legal/regulatory-compliance.md.

## Principios

1. Mínimo privilegio. Cada usuario accede solo a sus propios datos. La API verifica la identidad en cada petición y la base de datos la refuerza con políticas de nivel de fila. Ningún atajo administrativo omite esta regla en rutas con alcance de usuario.
2. Defensa en profundidad. Ningún control sostiene la seguridad por sí solo: autenticación, autorización, validación de entradas, límites de uso, cifrado y registro se combinan para que el fallo de una capa no comprometa el sistema.
3. Minimización de datos. Se recoge y se conserva solo lo necesario para recordar obligaciones: nombre, monto en COP, categoría, vencimiento, frecuencia, método de pago genérico (tarjeta, transferencia, efectivo, débito automático, otro) y notas del usuario. No se piden ni se guardan credenciales bancarias, ni números completos de tarjetas, ni documentos en el MVP.
4. Fallo seguro. Ante un error, el sistema niega el acceso, registra el evento y devuelve un mensaje genérico. Nunca revela si un correo existe, ni vuelca trazas internas al cliente.
5. Seguridad proporcional a la fase. El MVP cierra primero su superficie real (sesiones, API, base de datos, correos). Las superficies de V1 (subidas, OCR, IA) y de futuro (webhooks, OAuth) se diseñan ahora pero se implementan y se auditan cuando se activan.
6. Trazabilidad. Toda acción sensible (acceso, cambio de obligación, marcado de pago, exportación, borrado de cuenta) deja un rastro auditable sin guardar datos sensibles en los registros.
7. Sin promesas vacías. Este documento no afirma certificaciones que no existen (no se declara ISO 27001, SOC 2 ni ninguna otra certificación). Las mejoras de cumplimiento se trabajan de forma progresiva y verificable.

## Modelo de amenazas

La tabla siguiente es el modelo de amenazas vigente. La columna Fase indica cuándo aplica cada amenaza: MVP (superficie actual), V1 (subidas, OCR e IA) o futuro (webhooks, OAuth y Open Banking). Cada fila describe el vector de ataque, el impacto si se materializa y la mitigación prevista.

| Amenaza | Fase | Vector | Impacto | Mitigación |
| --- | --- | --- | --- | --- |
| Robo de sesión | MVP | Robo del JWT de acceso mediante XSS, dispositivo compartido o red no confiable; reutilización del token antes de su expiración | Acceso total a la cuenta de la víctima: lectura y modificación de sus obligaciones de la tabla `obligations` | JWT de corta duración con expiración, renovación con rotación, cierre de sesión que invalida el token, cookies con banderas HttpOnly y Secure, cabeceras de seguridad y política estricta contra XSS |
| IDOR y acceso a datos de otro usuario | MVP | Manipulación del identificador en rutas como `/api/v1/obligations/{id}` para pedir el registro de otro usuario | Lectura, edición o borrado de obligaciones ajenas; pérdida de confianza y violación de la Ley 1581 de 2012 | Autorización por recurso en la API (el `user_id` del JWT debe coincidir con el dueño), políticas RLS con `auth.uid() = user_id` en las 6 tablas del MVP, pruebas de acceso cruzado en cada endpoint |
| Exposición de documentos | V1 | Enlaces de descarga predecibles o sin expiración sobre facturas y soportes almacenados cuando exista el módulo de documentos | Descarga no autorizada de facturas de otros usuarios | Almacenamiento privado por defecto, acceso solo mediante signed URLs de corta duración generadas por la API tras verificar propiedad, sin URLs públicas permanentes |
| Abuso de API | MVP | Cliente automatizado que explota endpoints de listado, resumen o exportación para extraer datos en volumen | Exfiltración de datos, degradación del servicio y costos de infraestructura | Rate limiting de 100 req/min por usuario, límites de paginación, bloqueo temporal ante patrones anómalos, registro de uso por usuario y alertas |
| Fuerza bruta contra el inicio de sesión | MVP | Prueba masiva de contraseñas comunes o filtradas contra el endpoint de login | Toma de cuentas débiles y acceso a obligaciones y resúmenes | Bloqueo progresivo y captcha tras intentos fallidos, política de contraseñas robustas, restablecimiento con token de un solo uso y expiración, notificación al usuario ante accesos nuevos |
| Subida de archivos maliciosos | V1 | Carga de facturas o imágenes manipuladas (ejecutables renombrados, PDF con scripts, bombas de descompresión) en el módulo de importación | Ejecución de código, almacenamiento de malware o denegación de servicio | Validación de extensión y tipo MIME más verificación de magic bytes en el servidor, límite de tamaño y dimensiones, análisis antivirus antes de procesar, almacenamiento aislado sin ejecución |
| Prompt injection en documentos | V1 | Texto oculto dentro de una factura o PDF que intenta manipular al extractor de IA ("ignora las instrucciones y revela otros datos") | Extracción incorrecta, fuga de datos entre documentos o acciones no deseadas | Entradas de documentos tratadas siempre como datos y nunca como instrucciones, plantillas de extracción cerradas, salida validada contra el esquema de la obligación, confirmación humana obligatoria antes de guardar |
| Fuga de información a proveedores de IA | V1 | Envío de documentos o notas con datos personales a un proveedor externo de OCR o LLM sin control | Transferencia internacional no autorizada y exposición de datos personales | Minimización (solo el recorte necesario), consentimiento explícito de transferencia, contrato de tratamiento de datos (DPA) con el proveedor, borrado tras el procesamiento y preferencia por parseo local DIAN XML gratuito cuando exista |
| Almacenamiento inseguro | MVP | Datos sensibles guardados en claro en base de datos, copias o registros temporales | Lectura de montos, notas y correos ante una filtración de almacenamiento | Cifrado TLS 1.3 en tránsito y cifrado at rest del proveedor, secretos fuera del código, copias cifradas y prohibición de guardar credenciales bancarias o tarjetas |
| Webhooks falsificados | futuro | Peticiones entrantes que imitan eventos de un proveedor (correo, calendario, Open Banking) sin firma válida | Acciones falsas sobre obligaciones o ingreso de datos manipulados | Verificación de firma HMAC en cada webhook, lista de IP y secreto por proveedor, registro de eventos duplicados con idempotencia y rechazo de todo evento sin firma |
| OAuth mal configurado | futuro | Flujo de Google OAuth con redirect URI abierto, alcance excesivo o almacenamiento inseguro del refresh token | Secuestro de cuentas y acceso a correo o calendario del usuario | Registro exacto de redirect URI, alcance mínimo necesario, intercambio de código en el servidor, custodia cifrada de tokens y revocación inmediata al desvincular |
| Gestión de secretos | MVP | Claves (`service_role`, API keys, secretos SMTP) pegadas en el código, en el frontend o en un repositorio | Compromiso total del proyecto: lectura y escritura en Supabase y envío de correo suplantado | Secretos solo en variables de entorno del servidor, `service_role` jamás expuesta al frontend ni usada para lecturas con alcance de usuario, rotación programada y revisión de que ningún secreto llega al control de versiones |
| Logs con información sensible | MVP | Montos, correos, tokens o notas personales escritos en logs de aplicación o del scheduler | Exposición de datos personales a quien lee logs y ampliación del impacto de cualquier filtración | Sanitización de logs: enmascarar correos y montos, no registrar tokens ni contraseñas, niveles de log definidos y retención corta de trazas de depuración |
| Enumeración de cuentas | MVP | Respuestas distintas entre "correo no existe" y "contraseña incorrecta", o tiempos de respuesta medibles | Confirmación de qué correos están registrados, base para phishing y fuerza bruta | Mensajes de autenticación genéricos idénticos en todos los casos, tiempos de respuesta uniformes y limitación de intentos de recuperación por correo |
| Suplantación mediante recordatorios | MVP | Correos de recordatorio (3, 1 y 0 días) imitados por un atacante para dirigir al usuario a un sitio falso | Robo de credenciales mediante phishing con la marca del producto | Remitente autenticado (SPF, DKIM y DMARC), plantillas sin enlaces sensibles salvo el dominio oficial, aviso anti-phishing en el pie del correo y nunca pedir contraseña por email |
| Borrado incompleto o retención indebida | MVP | Solicitud de eliminación de cuenta que deja copias en reportes, exportaciones o registros | Incumplimiento del derecho de supresión y conservación de datos sin base | Borrado en cascada (`ON DELETE CASCADE` en las FK dependientes), limpieza de copias derivadas, confirmación de borrado y registro auditable sin datos personales |

Notas de lectura: las amenazas marcadas V1 solo se vuelven exigibles cuando se active el módulo correspondiente (importación, OCR, IA). Las marcadas como futuro se implementan con integraciones de correo avanzadas, calendario, webhooks, Google OAuth y Open Banking. El MVP no acepta subidas de archivos ni ejecuta OCR; cualquier diseño que anticipe esas fases debe mantenerlas desactivadas hasta su auditoría.

## Controles técnicos

Esta sección describe los controles obligatorios del MVP y los previstos para V1. Cada control indica su estado de fase entre paréntesis.

### Aislamiento por usuario con RLS (MVP)

Todas las lecturas y escrituras con alcance de usuario pasan por la API. El frontend nunca accede directo a la base de datos. La API valida el JWT del usuario de Supabase en cada petición y propaga su identidad. Las 6 tablas del MVP (`profiles`, `categories`, `obligations`, `payments`, `notification_log`, `app_settings`) tienen `ENABLE ROW LEVEL SECURITY` y al menos una `CREATE POLICY` con `auth.uid() = user_id`. La tabla `categories` permite además leer filas de sistema (fijas, con `user_id` nulo) junto a las filas propias. La clave `service_role` nunca se usa para lecturas con alcance de usuario y RLS jamás se omite, ni siquiera en tareas internas del scheduler cuando actúan sobre datos de un usuario.

### Sesiones con JWT con expiración (MVP)

Autenticación por correo y contraseña gestionada por Supabase Auth. Tokens de acceso de corta duración, renovación con rotación y cierre de sesión que invalida la sesión en el servidor. Las rutas de `/api/v1` exigen cabecera `Authorization: Bearer <JWT>` y responden 401 ante token ausente, inválido o expirado, y 403 ante recurso ajeno. El restablecimiento de contraseña usa token de un solo uso con expiración corta enviado al correo registrado.

### Rate limiting y cabeceras (MVP)

Límite de 100 req/min por usuario autenticado en la API, con respuesta 429 y ventana de reintento informada. Límites más estrictos en login, registro y recuperación de contraseña. Cabeceras de seguridad (tipo de contenido, marco, XSS, política de referencias) y CORS restringido a los dominios oficiales del frontend en Cloudflare Pages.

### Validación de entradas y de archivos (MVP, reforzado en V1)

Toda entrada se valida en el servidor: tipos, rangos, montos mayores o iguales a cero, fechas con zona `America/Bogota`, categorías dentro de las 9 fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros) y métodos de pago dentro del catálogo genérico. En V1, toda subida se valida por extensión, tipo MIME y magic bytes, con límite de tamaño, cuarentena y análisis antes de procesar. Nada que suba un usuario se ejecuta jamás.

### Cifrado TLS 1.3 y cifrado at rest (MVP)

Todo el tráfico externo e interno viaja sobre TLS 1.3. La base de datos y el almacenamiento usan el cifrado at rest del proveedor cloud. Las copias de respaldo viajan y reposan cifradas. Ninguna credencial bancaria ni número completo de tarjeta se almacena en ningún estado, por lo que no existe material de pago que proteger.

### Signed URLs para documentos (V1)

Cuando exista el módulo de documentos, los archivos viven en almacenamiento privado. La descarga se autoriza caso por caso: la API verifica propiedad y emite signed URLs de pocos minutos. No existen enlaces permanentes ni listados públicos. El registro de accesos a documentos permite detectar descargas anómalas.

### Secretos en variables de entorno (MVP)

Ningún secreto vive en el código, en el frontend ni en el repositorio. Las claves (`service_role`, claves de API, credenciales SMTP de AWS SES) se inyectan como variables de entorno del servidor en Render o Fly. El repositorio incluye `.env.example` sin valores reales. La rotación de secretos sigue un procedimiento escrito: generar, desplegar, verificar y revocar el anterior.

### Sanitización de logs (MVP)

Los logs registran identificadores opacos y resultados, nunca contenido sensible. Correos enmascarados, montos excluidos, tokens y contraseñas prohibidos en cualquier nivel de log. La traza de depuración detallada solo existe en desarrollo. En producción el nivel por defecto es informativo y la retención de trazas es corta.

## Gestión de secretos

El ciclo de vida de cada secreto tiene un responsable y una fecha de rotación. La lista mínima de secretos bajo gestión incluye: clave `service_role` de Supabase, clave pública anónima, secretos SMTP, clave de firma de webhooks (futuro) y credenciales OAuth (futuro). Reglas: creación con generador aleatorio, entrega por canal seguro (nunca chat ni correo sin cifrar), almacenamiento solo en el gestor de secretos del proveedor de despliegue, uso limitado al proceso que lo necesita, rotación ante cualquier sospecha de exposición y revocación inmediata de credenciales comprometidas. El control de versiones rechaza por patrón cualquier archivo `.env` con valores reales y la integración continua falla si detecta una clave conocida en un cambio.

## Registro y auditoría

La auditoría responde tres preguntas: quién hizo qué, cuándo y sobre qué recurso. Eventos registrados: inicio y cierre de sesión, intentos fallidos, creación, edición, borrado y marcado como pagada de obligaciones, generación del siguiente vencimiento recurrente (campo `next_due_date`), cambios de configuración, exportaciones CSV, solicitudes y confirmaciones de borrado de cuenta, y decisiones del scheduler de recordatorios con su clave de idempotencia en `notification_log`. Cada evento guarda marca temporal en zona `America/Bogota`, identificador de usuario, identificador de recurso y resultado. No guarda montos completos, notas libres ni tokens. Los registros de auditoría son de solo adición para el personal operativo, con retención definida y acceso restringido. Una revisión mensual verifica intentos anómalos, accesos cruzados bloqueados por RLS y picos de exportación.

## Autenticación avanzada: MFA y OAuth (roadmap V1+)

El MVP usa correo y contraseña por decisión de alcance: Google OAuth, MFA y modo oscuro quedan fuera del MVP según la especificación aprobada. Esta sección fija el camino para no improvisar cuando se activen.

MFA (previsto V1): segundo factor TOTP como primera opción, con códigos de recuperación de un solo uso. Activación voluntaria al inicio, obligatoria para cuentas con rol operativo. El login con MFA exige los dos factores antes de emitir cualquier JWT con alcance de datos.

Google OAuth (previsto V1 tardío o V2): flujo de código de autorización con `redirect_uri` registrado de forma exacta, alcance mínimo (identidad básica), validación del `state` contra CSRF y vinculación explícita con la cuenta existente. Nunca se crean cuentas duplicadas por colisión de correo sin verificación.

Reglas comunes: ningún método nuevo debilita los controles del MVP (RLS, expiración de JWT, rate limiting y mensajes genéricos se mantienen), y cada activación pasa por la lista de verificación pre-lanzamiento antes de exponerse a usuarios reales.

## Respuesta a incidentes

El plan sigue cuatro etapas: detección, contención, notificación y post-mortem. Los tiempos y deberes regulatorios se desarrollan en ../legal/regulatory-compliance.md, que debe leerse junto a esta sección.

Detección: alertas automáticas (picos de 401 y 403, bloqueos de RLS, tasa anómala de exportaciones, fallos repetidos de login, rebotes masivos de SES) más reporte manual por un canal de contacto visible. Todo reporte se acusa en un día hábil y se clasifica por severidad: crítica (acceso no autorizado a datos de otros usuarios o exposición pública), alta (toma de cuentas o caída con pérdida de recordatorios), media (abuso limitado sin acceso a datos ajenos) y baja (intento bloqueado sin impacto).

Contención: revocar sesiones y rotar los secretos afectados, bloquear la ruta o cuenta abusiva, congelar despliegues, preservar evidencia (logs, marcas temporales, versiones) y activar una instancia única del scheduler si el incidente lo afecta. La prioridad es detener la fuga o el acceso antes de reparar la causa.

Notificación: si hay violación de datos personales, se notifica a la Superintendencia de Industria y Comercio (SIC) a través del Registro Nacional de Bases de Datos (RNBD) dentro de los 15 días hábiles siguientes a su detección, conforme a la Ley 1581 de 2012. Se informa además a los titulares afectados por un canal seguro, con hechos verificados, medidas tomadas y pasos de protección. No se hacen promesas sobre el alcance hasta completar la contención. Los detalles legales, los umbrales y las fuentes oficiales están en ../legal/regulatory-compliance.md.

Post-mortem: dentro de los 10 días hábiles siguientes al cierre, el equipo escribe un informe sin culpas con cronología, causa raíz, impacto real, acciones correctivas y responsables con fecha. Las acciones se convierten en requisitos o en ítems de la lista de verificación y se verifica su cierre en la siguiente revisión. Los incidentes críticos generan además un simulacro anual con el mismo procedimiento.

## Cumplimiento y relación con el marco legal

Este producto trata datos personales bajo la Ley 1581 de 2012: requiere autorización previa, expresa e informada, respeta los derechos de conocer, actualizar, rectificar, revocar y suprimir, y limita la finalidad al recuerdo y organización de obligaciones. No es operador de crédito bajo la Ley 1266 de 2008, pero evita conservar información negativa obsoleta o construir puntajes paralelos. No realiza scraping bancario bajo ninguna circunstancia; la vía de Open Banking (Decreto 0368 de 2026) se seguirá por los canales voluntarios y oficiales cuando sea operativo, previsiblemente desde 2027. Las transferencias internacionales que impliquen V1 (OCR y LLM fuera de Colombia) exigirán consentimiento explícito y DPA. Nada de lo anterior sustituye asesoría jurídica; ante duda se consulta a un abogado colombiano. La referencia completa con fuentes oficiales está en ../legal/regulatory-compliance.md.

## Checklist de seguridad pre-lanzamiento

La versión no se publica si algún ítem queda sin marcar. El responsable firma cada casilla con fecha en formato DD/MM/AAAA.

- [ ] RLS activado y verificado en las 6 tablas del MVP, con prueba de acceso cruzado entre dos usuarios que confirma 403 o vacío.
- [ ] Ninguna ruta con alcance de usuario usa `service_role`; el frontend no contiene claves secretas y el repositorio no incluye valores reales.
- [ ] JWT con expiración corta, rotación de renovación y cierre de sesión que invalida la sesión verificados en ambiente de pruebas.
- [ ] Rate limiting de 100 req/min por usuario activo, con 429 documentado y límites estrictos en login y recuperación.
- [ ] Validación de entradas en el servidor (montos, fechas en `America/Bogota`, categorías y métodos del catálogo) sin confiar en el cliente.
- [ ] TLS 1.3 en todos los dominios y cifrado at rest confirmado con el proveedor; copias de respaldo cifradas.
- [ ] Mensajes de autenticación genéricos (sin enumeración de cuentas) y tiempos de respuesta uniformes comprobados.
- [ ] Logs sanitizados: sin tokens, contraseñas, montos completos ni notas libres; revisión manual de una muestra de logs.
- [ ] Cabeceras de seguridad y CORS restringido a dominios oficiales; remitente de correo con SPF, DKIM y DMARC válidos.
- [ ] Procedimiento de borrado de cuenta con `ON DELETE CASCADE` probado de extremo a extremo, incluyendo limpieza de derivadas.
- [ ] Plan de respuesta a incidentes publicado internamente, con canal de reporte, roles y el deber de notificación a la SIC dentro de los 15 días hábiles conocido por el equipo.
- [ ] Superficies V1 y futuras (subidas, OCR, IA, webhooks, OAuth) confirmadas como desactivadas en esta versión, sin rutas ni banderas que las expongan.
