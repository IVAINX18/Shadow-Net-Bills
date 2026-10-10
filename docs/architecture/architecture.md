# Arquitectura del sistema Shadow-Net Bills

## 1. Resumen

Shadow-Net Bills es una aplicacion web para organizar y recordar obligaciones financieras personales en Colombia. El MVP sigue un diseno manual-primero: el usuario registra sus obligaciones a mano, el sistema envia recordatorios por correo y presenta un calendario mensual con un resumen de gastos. No hay integraciones bancarias, no se mueve dinero y no hay OCR ni IA en esta fase.

La arquitectura del MVP es un monolito modular desplegado como dos piezas: un frontend estatico servido por CDN y una API unica que concentra la logica, la programacion de recordatorios y el acceso a datos. La persistencia y la autenticacion las provee Supabase (PostgreSQL con RLS mas Auth), y el envio de correos usa AWS SES. Esta combinacion minimiza piezas moviles, mantiene el costo mensual entre $30 y $55 USD para 100 a 500 usuarios, y permite que un equipo pequeno opere el sistema sin personal dedicado a infraestructura.

El documento describe componentes, flujos, decisiones justificadas, continuidad del programador de tareas, reglas de acceso a datos, limites del MVP, preparacion para V1 y despliegue. El detalle de tablas vive en `../architecture/database.md`, la especificacion de endpoints en `../architecture/api.md`, el stack justificado en `../technical/stack-selection.md` y los costos en `../technical/infrastructure.md`.

## 2. Diagrama general

El trafico entra por el CDN, llega a la API, y la API habla con Supabase y con SES. El programador de tareas corre dentro de la API.

```
                        HTTPS                     HTTPS                    SQL + Auth
  +----------------+  ---------->  +------------------------+  -------------------->  +-------------------+
  | Navegador del  |                | Cloudflare Pages       |                          | Supabase          |
  | usuario        |  ---------->  | (frontend estatico:    |                          | - Auth (JWT)      |
  +----------------+   estatico     |  React + TS + Vite)    |                          | - PostgreSQL + RLS|
                                    +-----------+------------+                          | - Storage (V1)    |
                                                |                                       +---------+---------+
                                                | API REST                              |         ^
                                                v                                       v         |
                                    +------------------------+                +-------------------+
                                    | API FastAPI            |  ---- JWT ----> | Valida JWT de     |
                                    | en Render o Fly        |   verifica     | Supabase Auth     |
                                    |                        |                +-------------------+
                                    | + APScheduler          |
                                    |   embebido (un proceso)|  SMTP/API
                                    | + cliente Supabase     | ---------->  +-------------------+
                                    |   (unico acceso a DB)  |              | AWS SES           |
                                    +------------------------+              | (correos 3,1,0)   |
                                                                            +-------------------+
```

Diagrama de despliegue logico (solo ASCII):

```
  [Cloudflare Pages] --HTTPS--> [API FastAPI en Render o Fly] --SQL/RLS--> [Supabase Postgres]
                                   |        |                         |
                                   |        +--SMTP--> [AWS SES]       +--Auth--> [Supabase Auth]
                                   |
                                   +--[APScheduler embebido: job store en Postgres o pg_cron]
```

Las flechas se leen asi: el frontend solo llama a la API por HTTPS con Bearer JWT; la API valida el token y consulta Supabase con la identidad del usuario; el programador despierta cada cierto intervalo, calcula vencimientos y envia correos via SES registrando cada envio en `notification_log`.

## 3. Componentes

### 3.1 Frontend en Cloudflare Pages

Aplicacion de pagina unica en React con TypeScript y Vite. Es contenido estatico (HTML, JS, CSS) distribuido por CDN, sin servidor propio ni secretos. Guarda el JWT de sesion en memoria o almacenamiento local del navegador y lo envia como cabecera `Authorization: Bearer <token>` en cada llamada. No conoce cadenas de conexion ni claves de servicio. Sus responsabilidades son presentacion, validacion basica de formularios y manejo de estados de carga, vacio y error.

### 3.2 API FastAPI en Render o Fly

Proceso unico en Python con FastAPI que expone la API REST bajo la base `/api/v1/obligations` y rutas hermanas (categorias, resumen, calendario, configuracion, exportacion, cuenta). Responsabilidades: validar entradas, verificar el JWT contra Supabase Auth, aplicar reglas de negocio (recurrencias, calculo de `next_due_date`, offsets de recordatorio `{3,1,0}` dias), leer y escribir en la tabla `obligations` y sus vecinas, y exponer el programador de recordatorios. Se despliega como una sola instancia web en Render (plan de ~$25 USD) o Fly (maquinas de ~$4 a $7 USD) en la region us-east-1.

### 3.3 Supabase: Auth y PostgreSQL

Supabase provee dos servicios: Auth (registro, login con correo y contrasena, emision y verificacion de JWT, recuperacion de contrasena) y PostgreSQL con Row Level Security. Las seis tablas del MVP son `profiles`, `categories`, `obligations`, `payments`, `notification_log` y `app_settings`. Toda fila de usuario lleva `user_id` y sus politicas exigen `auth.uid() = user_id`. La tabla `categories` mezcla filas de sistema (`user_id` nulo, las 9 categorias fijas: Vivienda, Servicios publicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educacion, Gobierno, Otros) con filas propias de cada usuario. Storage se reserva para V1 (documentos e imagenes de facturas).

### 3.4 APScheduler embebido

Libreria de programacion que corre dentro del mismo proceso de la API. Cada hora revisa las obligaciones con `reminder_enabled = true`, calcula los dias restantes hasta `next_due_date` y encola envios para los offsets `{3,1,0}`. Usa un job store persistente en la misma base de datos (o `pg_cron` como alternativa descrita en la seccion 6) para no perder la agenda ante reinicios. No requiere un servicio aparte ni infraestructura de colas en el MVP.

### 3.5 AWS SES para correos

Servicio de envio de correo transaccional (~$0.10 USD por cada 1.000 correos). La API lo invoca solo desde el lado servidor con credenciales guardadas como variables de entorno. Cada recordatorio genera una fila en `notification_log` con clave unica por obligacion, tipo y fecha, de modo que un reintento no duplica correos.

## 4. Flujos principales

### 4.1 Crear obligacion

1. El usuario completa el formulario (nombre, monto en COP, categoria entre las 9 fijas, fecha de vencimiento, frecuencia entre one_time, weekly, monthly, quarterly y yearly, metodo generico entre tarjeta, transferencia, efectivo, debito automatico u otro, y notas opcionales).
2. El frontend envia `POST /api/v1/obligations` con el Bearer JWT.
3. La API valida el JWT, valida los campos (monto mayor o igual a cero, categoria permitida, fecha valida) y calcula `next_due_date` igual a la fecha ingresada.
4. La API inserta la fila en `obligations` con `user_id` del token y `status = pendiente`.
5. La API responde 201 con el objeto creado; el frontend lo muestra en la lista y en el calendario.

### 4.2 Recordatorio por programador

1. APScheduler despierta segun su intervalo (cada hora en el MVP).
2. Consulta obligaciones con `reminder_enabled = true`, `status = pendiente` y `next_due_date` dentro de 3, 1 o 0 dias.
3. Por cada candidata verifica `notification_log`: si ya existe fila con la misma obligacion, tipo y fecha, la salta (idempotencia).
4. Llama a AWS SES con asunto y cuerpo en espanol, incluyendo nombre, monto con formato $20.000 y dias restantes.
5. Inserta la fila en `notification_log`; si el envio falla, no inserta y reintenta en el siguiente ciclo.
6. El usuario recibe hasta tres correos por vencimiento (3 dias antes, 1 dia antes, el mismo dia).

### 4.3 Marcar pago

1. El usuario pulsa "marcar como pagada" en una obligacion pendiente.
2. El frontend envia `POST /api/v1/obligations/{id}/pay`.
3. La API valida propiedad (RLS mas chequeo de `user_id`), inserta una fila en `payments` con monto y fecha, y actualiza la obligacion: si es `one_time`, pasa a `pagada`; si es recurrente, calcula el siguiente `next_due_date` segun la frecuencia y la deja `pendiente` para el proximo ciclo.
4. La API responde con la obligacion actualizada y el registro de pago; el frontend refresca lista, calendario y resumen mensual.

### 4.4 Login

1. El usuario envia correo y contrasena al endpoint de Auth.
2. Supabase Auth verifica credenciales y devuelve un JWT de acceso con expiracion corta mas un token de refresco.
3. El frontend guarda los tokens y adjunta el JWT en cada llamada posterior.
4. La API valida firma y expiracion del JWT en cada request antes de tocar la base; si expiro, responde 401 y el frontend usa el token de refresco o redirige al login.

## 5. Decisiones de arquitectura con justificacion

### 5.1 Monolito modular frente a arquitectura de servicios distribuidos

Decision: un solo servicio API con modulos internos (rutas, servicios, trabajos programados). Razon: el MVP tiene un dominio pequeno (seis tablas, una decena de endpoints) y un equipo pequeno; dividirlo en servicios separados multiplicaria despliegues, contratos y monitoreo sin beneficio medible. El monolito permite refactorizar por dentro y extraer un servicio en V1 o V2 solo cuando una parte duela de verdad (por ejemplo OCR). Costo: menor factura y menor operacion.

### 5.2 Supabase frente a infraestructura a medida en AWS

Decision: Supabase Pro (~$25 USD por mes) para Auth mas PostgreSQL con RLS. Razon: entrega autenticacion, base administrada, respaldos y politicas por fila sin codigo adicional; montarlo a medida (base administrada mas Auth0 o Cognito mas respaldos) costaria mas y tardaria semanas. La contrapartida es acoplarse a un proveedor, mitigada porque el acceso pasa por una capa fina (ver seccion 8) y el SQL es PostgreSQL estandar y portable.

### 5.3 Sin SQLAlchemy: cliente Supabase como estrategia unica de acceso

Decision: todo acceso a datos usa el cliente oficial de Supabase desde la API; no se introduce SQLAlchemy ni ninguna segunda estrategia. Razon tecnica explicita: dos mapeadores conviviendo generan dos fuentes de verdad para esquemas, transacciones y politicas, y duplican los lugares donde un error puede omitir RLS. Una sola estrategia mantiene un unico camino auditado: API valida JWT, cliente suplanta al usuario, PostgreSQL aplica RLS. Las migraciones de esquema se versionan como SQL en el repositorio.

### 5.4 APScheduler embebido frente a trabajador separado con colas

Decision: APScheduler dentro del proceso API con job store persistente. Razon: el volumen del MVP (cientos de usuarios, tres correos por vencimiento) cabe en un ciclo horario simple; un trabajador separado con colas exigiria otro servicio, otra cola y monitoreo extra. Cuando el volumen supere un umbral definido (miles de envios diarios o necesidad de reintentos con retroceso), se migra a un trabajador dedicado sin cambiar la interfaz de notificaciones.

### 5.5 Sin piezas distribuidas en el MVP

Decision: en el MVP no hay Redis, ni colas de mensajes distribuidas, ni orquestadores de contenedores (K8s), ni mallas de servicios. Razon: cada pieza suma costo fijo y superficie operativa; ninguna es necesaria para los limites del MVP (ver seccion 7). La sesion la sostiene el JWT sin estado, la agenda la sostiene la base de datos, y el despliegue es un contenedor unico por entorno. Se reevaluan solo con metricas de escala en V2.

## 6. Continuidad del programador

La seccion mas riesgosa del diseno es que los recordatorios no se pierdan ni se dupliquen ante reinicios o despliegues. Tres reglas lo garantizan.

### 6.1 Persistencia de la agenda

El programador no guarda su agenda en memoria. Usa un job store en la misma base PostgreSQL: cada tarea programada (nombre, proxima ejecucion, argumentos) vive en tabla, de modo que un reinicio del proceso recupera la agenda intacta. Como alternativa documentada, `pg_cron` (extension de Postgres que ejecuta SQL programado dentro de la base) puede disparar una funcion que invoque el endpoint interno de barrido; ambas opciones cumplen la misma propiedad: la agenda sobrevive al proceso. La eleccion por defecto es job store en base por simplicidad (una sola tecnologia), con `pg_cron` como plan B si el hospedaje limita procesos largos.

### 6.2 Restriccion de instancia unica

El programador debe correr en una sola instancia a la vez. Si la plataforma escala a dos replicas, ambas despertarian y enviarian correos duplicados. Por eso el despliegue del MVP fija replicas en uno para el proceso que contiene al scheduler (restriccion de instancia única), o bien designa un lider mediante bloqueo advisory de Postgres (`pg_advisory_lock`) donde solo quien obtiene el bloqueo ejecuta el barrido. La regla operativa es simple: instancia unica para el scheduler; escalar la API web sin scheduler solo cuando se separe el trabajador en V1.

### 6.3 Envio idempotente con notification_log

Aunque las dos reglas anteriores fallen un instante, el envio sigue siendo idempotente gracias a la tabla `notification_log`. Su clave unica cubre obligacion mas tipo de aviso mas fecha de envio (por ejemplo: obligacion 42, aviso "3d", fecha 15/10/2026). El flujo inserta despues de un envio exitoso y consulta antes de enviar; si dos procesos intentan el mismo aviso, uno choca con la restriccion unica y aborta sin reenviar. El cuerpo del correo incluye el identificador de la obligacion para rastreo, y los reintentos solo ocurren cuando no hay fila registrada.

## 7. Regla de acceso a Supabase

Cuatro mandamientos sin excepcion:

1. El frontend NUNCA accede directo a la base de datos. No recibe cadenas de conexion ni claves de servicio; solo llama a la API por HTTPS.
2. Toda lectura y escritura pasa por la API. La API valida el JWT del usuario de Supabase en cada request (firma, expiracion, usuario) y rechaza con 401 o 403 lo invalido o ajeno.
3. La clave `service_role` NUNCA se usa para lecturas o escrituras con alcance de usuario. Solo podria usarse en tareas administrativas del servidor explicitamente auditadas (por ejemplo purgas), jamas para servir datos de un usuario final, porque esa clave omite RLS.
4. RLS jamas se omite en el camino de usuario. Cada tabla del MVP tiene `ENABLE ROW LEVEL SECURITY` y politicas con `auth.uid() = user_id` (mas lectura de categorias de sistema con `user_id` nulo). El cliente de Supabase en la API suplanta al usuario autenticado para que Postgres aplique las politicas; ningun endpoint construye consultas que eludan el filtro por propietario.

Violar cualquiera de estas reglas es un defecto critico de seguridad y bloquea el lanzamiento.

## 8. Limites y supuestos del MVP

Supuestos operativos: 100 a 500 usuarios activos, hasta unas 15 obligaciones por usuario del plan gratuito, un ciclo de scheduler horario, y envio solo por correo (sin push, sin SMS). Limites tecnicos: una instancia de API con scheduler, sin cache distribuido, respaldos diarios de Supabase Pro, retencion de documentos fuera de alcance (Storage llega en V1), y exportacion CSV generada bajo demanda para volumenes pequenos. Supuesto de latencia: region us-east-1 por el corredor Bogota hacia Miami y Virginia del Norte, mas rapido desde Colombia que Sao Paulo. Supuesto de costo: ~$30 a $55 USD por mes totales; si el uso supera esos rangos, aplica la seccion de escalamiento de `../technical/infrastructure.md`. Fuera del MVP: OCR, modulos de suscripciones y cuotas como entidades propias, analitica, deteccion de cambios de precio, OAuth de Google, MFA, modo oscuro, lectura de correos, sincronizacion de calendario, iniciacion de pagos, banca abierta y app nativa.

## 9. Preparacion para V1 y siguientes (solo interfaces)

Para no bloquear el futuro sin construirlo ahora, el codigo de la API define tres interfaces abstractas sin implementaciones productivas en el MVP. Son contratos que V1 rellenara:

- `DocumentProcessor`: recibe un archivo o XML DIAN y devuelve datos extraidos con confianza (metodos: `parsear`, `extraer_campos`, `puntaje_confianza`). En MVP solo existe la definicion; V1 anadira OCR en la nube y parseo XML gratuito.
- `NotificationService`: envia avisos por un canal (metodos: `enviar_recordatorio`, `enviar_resumen`). En MVP la unica implementacion es correo via SES; V2 podra anadir push sin tocar los flujos.
- `AnalyticsService`: calcula agregados sobre obligaciones y pagos (metodos: `resumen_mensual`, `tendencia_gasto`). En MVP devuelve los agregados SQL simples del dashboard; V2 anadira estadistica y resmenes opcionales con modelos.

Ninguna tabla futura (`subscriptions`, `installments`, `documents`, `extracted_data`, `audit_logs`) se crea en el MVP; se listan como roadmap en `../architecture/database.md`.

## 10. Despliegue

Entornos: desarrollo (Supabase gratuito mas API local o preview) y produccion (Supabase Pro, API en Render o Fly, frontend en Cloudflare Pages, SES verificado con dominio propio). El despliegue es por contenedor unico para la API con variables de entorno para secretos (jamas en el repositorio), migraciones SQL versionadas aplicadas antes de cada release, y rollback por version anterior de imagen mas migracion reversible. Monitoreo con los niveles gratuitos (logs de la plataforma, chequeos de salud del endpoint, alertas de expiracion de certificados y de fallos de SES). Los detalles de precios, regiones y proyecciones de escala estan en `../technical/infrastructure.md` y la justificacion del stack en `../technical/stack-selection.md`. El siguiente paso tras aprobar esta especificacion es el desarrollo del MVP de 8 semanas descrito en el roadmap.
