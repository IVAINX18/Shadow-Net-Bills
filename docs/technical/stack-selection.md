# Selección de stack tecnológico

Estado: especificación aprobada para el MVP de Shadow-Net Bills. Fecha: 07/10/2026. Autor: Shadow-Net Technologies. Este documento define la pila tecnológica del MVP manual-primero (sin IA ni OCR en esta fase) y justifica cada decisión frente a sus alternativas.

## 1. Criterios de selección

La selección del stack sigue cinco criterios explícitos, aplicados a cada componente por igual. Primero, simplicidad operativa: el equipo debe poder desarrollar, probar y desplegar el MVP en 8 semanas sin personal dedicado a infraestructura. Segundo, costo predecible y bajo: el conjunto completo debe mantenerse dentro del rango de $30 a $55 USD al mes para 100 a 500 usuarios. Tercero, encaje con el dominio: autenticación por correo y contraseña, CRUD de obligaciones con recurrencia, recordatorios por correo en desfases de 3, 1 y 0 días, calendario mensual y exportación a CSV. Cuarto, seguridad por defecto: RLS nativo en base de datos, JWT con expiración, secretos en variables de entorno y TLS en tránsito y en reposo. Quinto, camino de evolución: cada pieza elegida permite crecer hacia V1 (OCR e importación) y V2 (análisis) sin reescribir la base, normalmente mediante interfaces abstractas ya previstas en la arquitectura.

Toda decisión se documenta con su alternativa evaluada, su razón y su costo. Cuando un precio depende del volumen o puede cambiar, se marca como [ESTIMACIÓN] con el supuesto escrito al lado.

## 2. Frontend: React con TypeScript y Vite

El frontend es una aplicación de página única con React, TypeScript y Vite, desplegada como sitio estático en Cloudflare Pages. El estado del servidor se gestiona con React Query y los estilos con Tailwind más shadcn/ui para componentes accesibles.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| React + TypeScript + Vite, SPA estática | Next.js con renderizado en servidor | La razón principal es que el MVP no necesita renderizado en servidor ni rutas dinámicas indexadas: son pantallas de aplicación tras autenticación. Vite ofrece arranque y construcción más rápidos y una salida estática simple que encaja con Cloudflare Pages. Next.js añadiría un servidor Node permanente sin beneficio para este caso | $0 al mes en Cloudflare Pages (plan gratuito) |

La alternativa Next.js se evaluó en serio porque facilita SEO y renderizado híbrido. Se descartó porque el producto no es un sitio público de contenidos: el valor está tras el inicio de sesión (panel, obligaciones, calendario, resumen). Mantener una SPA estática reduce la superficie operativa a un solo artefacto desplegado por cada cambio en la rama principal.

## 3. Backend: FastAPI con Python

La API REST vive en un monolito FastAPI con Python, bajo la base `/api/v1/obligations` y los demás recursos documentados en la especificación de API. FastAPI aporta validación con Pydantic, documentación OpenAPI automática y rendimiento suficiente para el volumen del MVP.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| FastAPI + Python + Pydantic, monolito | Django con su ORM y administración incluida | La razón es el tamaño del dominio: seis tablas y una veintena de endpoints no justifican el andamiaje completo de Django. FastAPI permite definir esquemas de petición y respuesta pegados a la especificación de API, con menos código ceremonial y pruebas directas con pytest y httpx | Incluido en el costo del alojamiento de la API (ver sección 9), sin licencia adicional |

Django se evaluó por su panel de administración y su ecosistema maduro. Se descartó porque su ORM introduciría una segunda estrategia de acceso a datos frente al cliente Supabase (ver sección 5), y porque el panel administrativo no es un requisito del MVP. La decisión mantiene una sola forma de leer y escribir datos en todo el backend.

## 4. Base de datos y autenticación: Supabase

Supabase provee PostgreSQL gestionado, autenticación con JWT, almacenamiento de objetos y políticas RLS nativas. El plan de referencia para el MVP es Supabase Pro a $25/mes, que aporta copias de respaldo diarias y ausencia de pausas por inactividad frente al nivel gratuito.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| Supabase (PostgreSQL + Auth + Storage + RLS) | PostgreSQL gestionado genérico + Auth0 para identidad | La razón es la integración: Auth, base de datos y RLS conviven en el mismo servicio, de modo que cada fila queda protegida por `auth.uid() = user_id` sin piezas intermedias. La alternativa obligaría a sincronizar usuarios entre Auth0 y la base propia y a replicar las políticas de aislamiento por usuario. Supabase además expone el almacenamiento con URLs firmadas para la fase V1 de documentos | $25/mes en plan Pro [ESTIMACIÓN según precios públicos de 2026]; nivel gratuito disponible solo para prototipo |

El RLS nativo es el control central contra el acceso entre usuarios (IDOR): cada una de las seis tablas del MVP tiene su política de fila. La API valida el JWT de Supabase en cada petición y nunca expone la clave `service_role` al frontend ni la usa para lecturas con alcance de usuario.

## 5. Acceso a datos: cliente Supabase, sin SQLAlchemy

Toda lectura y escritura desde el backend usa el cliente Supabase. No se usa SQLAlchemy ni ningún otro ORM o constructor de consultas en paralelo. Existe una sola estrategia de acceso a datos en el proyecto.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| Cliente Supabase como única vía de acceso | SQLAlchemy como ORM adicional junto al cliente | La razón técnica es evitar dos estrategias de acceso a la base de datos: dos modelos de sesión, dos formas de transacción y dos lugares donde auditar el aislamiento por usuario. Una sola estrategia significa que cada consulta atraviesa el mismo camino autenticado (JWT del usuario y RLS), lo que simplifica las revisiones de seguridad y los diagnósticos. Añadir SQLAlchemy duplicaría el mapeo de las seis tablas sin aportar consultas que el cliente no cubra en el MVP | $0 adicional; ahorro en mantenimiento al no duplicar modelos |

Dicho de forma directa: el proyecto funciona sin SQLAlchemy por decisión deliberada. El frontend nunca accede directo a la base; toda operación pasa por la API, y la API usa el cliente Supabase con el JWT del usuario. Las migraciones de esquema se versionan como SQL en el repositorio y se aplican de forma controlada.

## 6. Trabajos programados: APScheduler

Los recordatorios por correo (3, 1 y 0 días antes del vencimiento) y el resumen mensual se ejecutan con APScheduler embebido en el proceso de la API, con persistencia del calendario de trabajos en la misma base de datos y envío idempotente apoyado en la clave única de `notification_log`.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| APScheduler embebido con trabajos persistentes | Celery con trabajador y cola dedicados | La razón es la escala del MVP: unos pocos miles de trabajos diarios de correo no justifican operar un trabajador separado ni una cola persistente adicional. APScheduler, con su almacén de trabajos en la base existente y restricción de instancia única del planificador, cubre los desfases `{3,1,0}` y el resumen mensual con una pieza menos que vigilar. Celery queda como evolución si el volumen de V2 lo exige | $0 adicional; corre dentro del proceso de la API |

La alternativa Celery se evaluó por su madurez en cargas altas y reintentos distribuidos. Se descartó porque duplicaría la infraestructura mínima (proceso trabajador más respaldo de cola) para un volumen que cabe en el planificador embebido. La regla operativa queda fijada: el planificador no corre en dos réplicas a la vez, y cada envío registra su clave idempotente antes de llamar al proveedor de correo.

## 7. Correo electrónico: AWS SES

El envío de recordatorios y del resumen mensual usa AWS SES desde la región us-east-1. El costo de referencia es de aproximadamente $0,10 USD por cada 1.000 correos [ESTIMACIÓN según precios públicos de 2026], de modo que el MVP (decenas de miles de correos al mes) cuesta alrededor de $1 USD mensual.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| AWS SES | SendGrid | La razón es el costo por volumen bajo y la operación simple vía SMTP o API con dominio verificado y DKIM. SendGrid aporta plantillas visuales y analítica superiores, pero su plan de pago parte de unos $15 USD al mes [ESTIMACIÓN según precios públicos de 2026], innecesarios cuando los correos del MVP son transaccionales y de plantilla fija | SES ~$0,10 por 1.000 correos; SendGrid evaluado desde ~$15/mes |
| AWS SES | Mailgun | La razón frente a Mailgun es la misma: Mailgun es sólido en entregabilidad y registros, pero su plan básico parte de unos $15 USD al mes [ESTIMACIÓN según precios públicos de 2026] para volúmenes que SES cubre por centavos. Se mantiene SES mientras el correo sea transaccional; si V2 necesita campañas o pruebas A/B de contenido, se reevalúa | SES ~$0,10 por 1.000 correos; Mailgun evaluado desde ~$15/mes |

Ambas alternativas quedan documentadas para el futuro: si el equipo necesita editor de plantillas, analítica de apertura o gestión de campañas, SendGrid o Mailgun son los candidatos. Para el MVP, SES gana por costo y simplicidad.

## 8. IA y OCR: preparado, no incluido en el MVP

El MVP no incluye OCR ni modelos de lenguaje en ninguna ruta de usuario. La creación de obligaciones es manual, y la clasificación por categoría usa las 9 categorías fijas con reglas deterministas. La arquitectura, sin embargo, deja preparada la interfaz abstracta `DocumentProcessor` para que V1 conecte el conducto escalonado sin tocar los endpoints: primero análisis gratuito de XML DIAN, luego OCR en la nube a unos $10 USD por cada 1.000 páginas, y solo ante confianza baja (menor a 0,85) extracción con modelo ligero, siempre con confirmación humana obligatoria.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| Conducto preparado con interfaz abstracta, sin activar en MVP | Integrar OCR y LLM desde el primer lanzamiento | La razón es la validación: el riesgo del MVP es de demanda, no de tecnología de extracción. Activar OCR desde el día uno añadiría costo por factura, consentimientos de transferencia internacional y superficie de errores antes de saber si los usuarios crean obligaciones manualmente. Preparar la interfaz permite estimar el costo futuro (~$0,10 USD por usuario al mes a escala, según §C) sin pagarlo ahora | $0 en MVP; costo futuro documentado en la estrategia de IA |

Esta sección existe para impedir que la IA entre al MVP por la puerta trasera: ningún endpoint, trabajo programado o pantalla del MVP invoca proveedores de IA.

## 9. Alojamiento: Render o Fly.io para la API más Cloudflare Pages

La API FastAPI (con APScheduler embebido) se aloja en Render o en Fly.io, en la región us-east-1 por latencia desde Colombia. El frontend estático se sirve desde Cloudflare Pages con su CDN gratuita. Supabase aporta datos, Auth y almacenamiento.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| Render (plan básico) o Fly.io (instancia pequeña) + Cloudflare Pages | DigitalOcean App Platform o Droplet gestionado | La razón es el costo mínimo con despliegue desde el repositorio y certificados automáticos. DigitalOcean es una alternativa válida y predecible, pero su plataforma de aplicaciones o un droplet más base gestionada parten de unos $12 a $25 USD al mes [ESTIMACIÓN según precios públicos de 2026], por encima de Fly.io (~$4 a $7 al mes) para el mismo monolito pequeño | Render ~$25/mes o Fly.io ~$4 a $7/mes; Cloudflare Pages $0 |
| Render o Fly.io + Cloudflare Pages | AWS (EC2 o App Runner más RDS y balanceador) | La razón es la carga operativa: AWS ofrece control total, pero componer cómputo, base, balanceo, certificados y respaldos supera el presupuesto de operación de un MVP de 8 semanas. Se reevalúa cuando el tráfico o los requisitos de aislamiento lo justifiquen | AWS desde decenas de USD al mes según piezas [ESTIMACIÓN]; descartado para el MVP |

La combinación recomendada queda así: Supabase Pro $25 más API en Render (~$25) o en Fly.io (~$4 a $7) más Cloudflare Pages $0 más SES (~$1) más niveles gratuitos de observabilidad, para un total de $30 a $55 USD al mes en el rango de 100 a 500 usuarios.

## 10. Herramientas de desarrollo

El desarrollo usa integración continua con GitHub Actions, análisis y formato de Python con ruff, black y mypy, y pruebas con pytest. El frontend aplica su linter y su comprobación de tipos en el mismo flujo.

| Decisión | Alternativa considerada | Razón | Costo |
| --- | --- | --- | --- |
| GitHub Actions + ruff + black + mypy + pytest | Flujo manual o herramientas dispersas por desarrollador | La razón es la consistencia: cada cambio de rama ejecuta lint, tipos y pruebas antes de fusionar, y cada fusión a la rama principal despliega a los entornos de homologación y producción. Ruff unifica lint e importaciones con rapidez, black fija el formato, mypy sostiene el tipado que FastAPI y Pydantic aprovechan, y pytest cubre API, planificador y reglas de recurrencia | $0 (minutos gratuitos de Actions para el volumen del MVP) |

Las convenciones de ramas (`main`, `feat/*`, `fix/*`) y de mensajes de confirmación se describen en el documento de estructura de repositorio. Ninguna herramienta de esta sección crea infraestructura de producción.

## 11. Tabla resumen final

| Componente | Decisión | Alternativa principal descartada | Razón breve | Costo |
| --- | --- | --- | --- | --- |
| Frontend | React + TypeScript + Vite | Next.js | Sin necesidad de renderizado en servidor; salida estática simple | $0 (Cloudflare Pages) |
| Backend | FastAPI + Python (monolito) | Django | Menos ceremonial; evita un segundo acceso a datos | Incluido en API |
| Datos y Auth | Supabase Pro | PostgreSQL gestionado + Auth0 | Auth, RLS y Storage integrados | $25/mes |
| Acceso a datos | Cliente Supabase, sin SQLAlchemy | SQLAlchemy en paralelo | Una sola estrategia de acceso auditable | $0 |
| Trabajos | APScheduler embebido | Celery | Sin trabajador ni cola extra para este volumen | $0 |
| Correo | AWS SES | SendGrid / Mailgun | Centavos frente a ~$15/mes por volumen bajo | ~$1/mes |
| IA y OCR | Preparado, fuera del MVP | OCR desde el día uno | Validar demanda antes de pagar extracción | $0 en MVP |
| Alojamiento | Render o Fly.io + Cloudflare Pages | DigitalOcean / AWS | Menor costo y operación mínima | $4 a $25/mes |
| Dev tools | GitHub Actions, ruff, black, mypy, pytest | Flujo manual | Calidad repetible en cada cambio | $0 |
| Total MVP | Monolito + Supabase + Pages + SES | Infraestructura mayor | Simplicidad y costo predecible | $30 a $55/mes |
