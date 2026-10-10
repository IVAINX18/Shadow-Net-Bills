# Estructura de repositorio propuesta para Shadow-Net Bills

> Estado: propuesta para la siguiente fase (desarrollo del MVP). Fecha: 07/10/2026. Autor: Shadow-Net Technologies.

Este documento describe la estructura de directorios y archivos que se propone para el repositorio de Shadow-Net Bills cuando inicie la fase de desarrollo del MVP. Hoy el repositorio contiene únicamente especificación en `docs/` y no existe código de aplicación. Todo lo que sigue es una propuesta de organización, no una creación de directorios.

Para contexto de decisiones de arquitectura, ver [arquitectura del sistema](../architecture/architecture.md). Para decisiones de tecnología, ver [selección de stack](stack-selection.md) y para costos de infraestructura, ver [infraestructura y costos](infrastructure.md).

## 1. Principios de organización

La estructura propuesta sigue cinco principios simples que se mantienen durante todo el MVP.

Primero, separación clara por responsabilidad. El código del servidor vive en un solo lugar, el código del cliente en otro, y la base de datos y sus migraciones en un tercero. Ningún archivo pertenece a dos sitios a la vez. Si alguien duda dónde va un archivo nuevo, la respuesta se encuentra en la explicación por carpeta de este documento.

Segundo, monolito modular en el servidor. El MVP no usa microservicios. Existe una sola aplicación FastAPI organizada por capas internas (rutas, esquemas, servicios, integraciones, trabajos programados y utilidades). Cada capa tiene una responsabilidad y las dependencias apuntan siempre hacia adentro, nunca en círculos.

Tercero, una sola estrategia de acceso a datos. Toda lectura y escritura a la base de datos pasa por la API del servidor usando el cliente de Supabase. El cliente web nunca accede directo a la base. La clave de servicio nunca se usa para lecturas con alcance de usuario y las políticas de acceso por fila jamás se omiten. Este principio ya está definido en la arquitectura y la estructura de carpetas lo refuerza.

Cuarto, configuración fuera del código. Secretos, claves y URLs de entorno viven en variables de entorno y nunca en archivos versionados. Los archivos de ejemplo llevan el sufijo de ejemplo y contienen valores ficticios.

Quinto, todo cambio entra por control de versiones con revisión. La rama principal siempre está desplegable. Cada cambio llega por una rama corta con un propósito, pasa por revisión y por integración continua antes de integrarse.

## 2. Árbol completo propuesto

El siguiente bloque muestra el árbol completo propuesto para la siguiente fase. Es una referencia de organización. Los directorios `backend/`, `frontend/`, `supabase/`, `scripts/` y `.github/` orientados a código todavía no existen en el repositorio.

```text
shadow-net-bills/
├── README.md
├── docker-compose.yml
├── .gitignore
├── .github/
│   └── workflows/
│       ├── ci.yml
│       └── deploy.yml
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   ├── deps.py
│   │   │   ├── v1/
│   │   │   │   ├── __init__.py
│   │   │   │   ├── auth.py
│   │   │   │   ├── obligations.py
│   │   │   │   ├── categories.py
│   │   │   │   ├── dashboard.py
│   │   │   │   ├── calendar.py
│   │   │   │   ├── settings.py
│   │   │   │   ├── export.py
│   │   │   │   └── account.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   ├── config.py
│   │   │   ├── security.py
│   │   │   └── supabase_client.py
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   └── domain.py
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── obligation.py
│   │   │   ├── category.py
│   │   │   ├── payment.py
│   │   │   └── settings.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── obligation_service.py
│   │   │   ├── reminder_service.py
│   │   │   ├── summary_service.py
│   │   │   └── export_service.py
│   │   ├── integrations/
│   │   │   ├── __init__.py
│   │   │   ├── email_provider.py
│   │   │   └── interfaces.py
│   │   ├── jobs/
│   │   │   ├── __init__.py
│   │   │   └── scheduler.py
│   │   └── utils/
│   │       ├── __init__.py
│   │       ├── dates.py
│   │       └── money.py
│   └── tests/
│       ├── __init__.py
│       ├── conftest.py
│       ├── test_obligations.py
│       ├── test_reminders.py
│       └── test_export.py
├── frontend/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── index.html
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── components/
│       │   ├── ObligationCard.tsx
│       │   ├── ObligationForm.tsx
│       │   ├── MonthCalendar.tsx
│       │   └── SummaryCards.tsx
│       ├── pages/
│       │   ├── LoginPage.tsx
│       │   ├── OnboardingPage.tsx
│       │   ├── DashboardPage.tsx
│       │   ├── ObligationsPage.tsx
│       │   ├── CalendarPage.tsx
│       │   ├── SummaryPage.tsx
│       │   └── SettingsPage.tsx
│       ├── hooks/
│       │   ├── useObligations.ts
│       │   ├── useSummary.ts
│       │   └── useAuth.ts
│       ├── services/
│       │   ├── apiClient.ts
│       │   └── obligationsApi.ts
│       ├── stores/
│       │   └── sessionStore.ts
│       ├── types/
│       │   └── billing.ts
│       └── utils/
│           ├── formatCop.ts
│           └── dates.ts
├── supabase/
│   ├── migrations/
│   │   ├── 0001_profiles.sql
│   │   ├── 0002_categories.sql
│   │   ├── 0003_obligations.sql
│   │   ├── 0004_payments.sql
│   │   ├── 0005_notification_log.sql
│   │   └── 0006_app_settings.sql
│   └── functions/
│       └── health/
│           └── index.ts
├── docs/
│   ├── SHADOW_NET_BILLS_SPECIFICATION.md
│   ├── README.md
│   ├── product/
│   ├── architecture/
│   ├── ux/
│   ├── business/
│   ├── legal/
│   └── technical/
└── scripts/
    ├── dev_up.sh
    ├── seed_categories.sh
    └── check_links.sh
```

## 3. Carpeta backend: qué va y qué no va

La carpeta `backend/` contiene la aplicación FastAPI y sus pruebas. La subcarpeta `app/api/` contiene únicamente las rutas HTTP versionadas bajo la base `/api/v1/obligations` y sus módulos hermanos (autenticación, categorías, resumen, calendario, configuración, exportación y cuenta). Cada módulo de ruta valida la entrada, verifica el JWT del usuario y delega la lógica a la capa de servicios. Ninguna ruta accede a la base de datos por su cuenta.

La subcarpeta `app/core/` guarda la configuración por entorno, las utilidades de seguridad (verificación de JWT, límites de intentos) y el cliente único de Supabase. La subcarpeta `app/models/` guarda los tipos de dominio internos, mientras que `app/schemas/` guarda los esquemas de entrada y salida que definen el contrato JSON de la API. La subcarpeta `app/services/` contiene la lógica de negocio (obligaciones, recordatorios, resumen mensual, exportación a CSV). La subcarpeta `app/integrations/` contiene los adaptadores externos detrás de interfaces abstractas, como el proveedor de correo y las interfaces preparadas para la siguiente fase (`DocumentProcessor`, `NotificationService`, `AnalyticsService`). La subcarpeta `app/jobs/` contiene el programador de recordatorios con ejecución de instancia única y envío idempotente apoyado en la tabla `notification_log`. La subcarpeta `app/utils/` contiene ayudantes puros de fechas y formato de moneda.

La carpeta `tests/` contiene las pruebas automatizadas del servidor con sus accesorios de prueba. Los archivos `requirements.txt` y `Dockerfile` en la raíz de `backend/` fijan las dependencias y la imagen de ejecución.

Lo que no va en `backend/`: ningún componente visual, ningún secreto real, ninguna migración SQL (viven en `supabase/migrations/`), ninguna estrategia duplicada de acceso a datos y ningún servicio de infraestructura ajeno al MVP como colas distribuidas o mallas de servicios.

## 4. Carpeta frontend: qué va y qué no va

La carpeta `frontend/` contiene la aplicación web construida con React, TypeScript y Vite. La subcarpeta `src/components/` guarda piezas reutilizables de interfaz (tarjeta de obligación, formulario, calendario del mes, tarjetas de resumen). La subcarpeta `src/pages/` guarda una pantalla por ruta (acceso, incorporación, tablero, obligaciones, calendario, resumen y configuración). La subcarpeta `src/hooks/` guarda los ganchos de datos que consultan la API. La subcarpeta `src/services/` guarda el cliente HTTP centralizado que añade el token de portador y llama a la base `/api/v1/obligations`. La subcarpeta `src/stores/` guarda el estado de sesión. La subcarpeta `src/types/` guarda los tipos compartidos del dominio de facturación. La subcarpeta `src/utils/` guarda formateo de moneda en pesos (por ejemplo $20.000) y ayudantes de fecha.

Los archivos `package.json`, `vite.config.ts`, `tsconfig.json` e `index.html` en la raíz de `frontend/` definen dependencias, compilación, tipos y punto de entrada.

Lo que no va en `frontend/`: ninguna clave de servicio, ningún acceso directo a la base de datos, ninguna credencial bancaria, ninguna lógica de negocio que deba vivir en el servidor y ningún intento de mover dinero o iniciar pagos, porque Bills no mueve dinero por diseño.

## 5. Supabase, automatización, documentación y despliegue local

La carpeta `supabase/` guarda todo lo que pertenece a la plataforma de datos. La subcarpeta `migrations/` contiene una migración SQL numerada por cada una de las seis tablas del MVP (`profiles`, `categories`, `obligations`, `payments`, `notification_log`, `app_settings`), incluyendo índices, restricciones y políticas de acceso por fila. La subcarpeta `functions/` queda reservada para funciones de borde mínimas, como una comprobación de salud. No se diseña aquí ninguna tabla futura: suscripciones, cuotas, documentos y datos extraídos pertenecen a la hoja de ruta y solo se nombran en el modelo de datos.

La carpeta `scripts/` guarda guiones cortos de ayuda para desarrollo local (levantar el entorno, sembrar las nueve categorías fijas, verificar enlaces de documentación). La carpeta `docs/` conserva la especificación aprobada y seguirá siendo la fuente oficial durante el desarrollo. El archivo `docker-compose.yml` en la raíz permite levantar el servidor y sus dependencias para desarrollo local con un solo comando. La carpeta `.github/workflows/` guarda los flujos de integración y despliegue descritos en la sección siguiente. El archivo `README.md` en la raíz resume el proyecto, su estado y el enlace a la documentación.

## 6. Convenciones de nombres, ramas y commits

Los nombres de archivos usan minúsculas. El código Python usa minúsculas con guiones bajos y el código TypeScript usa mayúsculas iniciales para componentes y minúsculas iniciales con mayúsculas internas para funciones y variables. Las tablas y columnas de la base de datos usan minúsculas con guiones bajos, con la tabla `obligations` y su columna `next_due_date` como nombres consacrados. Las rutas de la API usan minúsculas, con guiones cuando hace falta, bajo la base `/api/v1/obligations`. Las categorías fijas conservan sus nombres oficiales (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros). Los métodos de pago genéricos se nombran con minúsculas (tarjeta, transferencia, efectivo, débito automático, otro).

Las ramas siguen un esquema corto. La rama `main` siempre está estable y desplegable. Las ramas de trabajo usan prefijos: `feat/` para funcionalidad nueva, `fix/` para correcciones, `docs/` para documentación y `chore/` para mantenimiento. Cada rama cubre un solo propósito y se integra pronto para evitar divergencias largas.

Los commits usan mensajes en español con prefijo convencional, por ejemplo `feat(api): agregar endpoint de resumen mensual` o `fix(web): corregir formato de moneda en el calendario`. Los cambios de documentación usan `docs(...)` y el mantenimiento usa `chore(...)`. Cada commit agrupa un solo tema y su mensaje explica qué cambió y por qué.

## 7. Integración y despliegue continuos propuestos

El flujo propuesto usa dos flujos automatizados en `.github/workflows/`. El flujo de integración (`ci.yml`) se ejecuta en cada solicitud de integración hacia `main`. Sus pasos son: instalar dependencias, verificar formato y tipos, ejecutar pruebas del servidor y del cliente, y comprobar que no existan secretos en el código. Si algún paso falla, la solicitud no se integra.

El flujo de despliegue (`deploy.yml`) se ejecuta solo cuando hay cambios integrados en `main`. Sus pasos son: construir las imágenes, aplicar migraciones pendientes en el entorno de producción, desplegar el servidor y publicar el cliente web. Las migraciones siempre avanzan hacia adelante y nunca editan una migración ya aplicada en producción. Los secretos de despliegue viven como secretos del repositorio y nunca en el código.

La estrategia de entornos es sencilla. Desarrollo local con `docker-compose.yml`, un entorno de pruebas compartido para revisión y un entorno de producción. Cada entorno tiene sus propias variables y su propia base de datos. Ningún dato real de producción se copia a entornos de desarrollo.

## 8. Relación con Shadow-Net Defender

Shadow-Net Bills y Shadow-Net Defender son productos hermanos dentro de la misma organización pero con repositorios y responsabilidades separadas. Bills organiza obligaciones financieras personales con recordatorios. Defender cubre otro dominio de seguridad que no se describe en este documento.

La relación entre ambos es de convivencia, no de dependencia. Comparten prácticas recomendadas (convenciones de commits, uso de integración continua, gestión de secretos por entorno) pero no comparten código de negocio ni base de datos. Si en el futuro aparece una pieza genuinamente común, se evaluará como una librería separada con su propio versionado. Durante el MVP no se crea ningún acoplamiento entre ambos productos y ningún cambio en Bills exige cambios en Defender.

## 9. Nota de fase: estructura propuesta, no crear directorios ahora

Esta sección es explícita para evitar malentendidos. La estructura descrita en este documento corresponde a la siguiente fase del proyecto, que es el desarrollo del MVP tras la aprobación de la especificación. Por favor no crear ahora ninguno de los directorios de código (`backend/`, `frontend/`, `supabase/functions/`, `scripts/` orientados a código) ni archivos de aplicación como manifiestos de dependencias, imágenes de contenedor o migraciones ejecutadas.

El repositorio debe permanecer en estado de planificación hasta que la especificación se apruebe. Crear esos directorios antes de tiempo generaría la impresión falsa de que el desarrollo ya empezó y complicaría la revisión de la especificación. Cuando la siguiente fase inicie, este documento servirá como plano para crear el árbol tal como se propone aquí.
