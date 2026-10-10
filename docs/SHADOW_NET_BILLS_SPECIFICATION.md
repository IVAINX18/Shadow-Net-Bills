# Especificación maestra de Shadow-Net Bills

**Estado del documento:** v1.0. **Fecha:** 07/10/2026. **Autor:** Shadow-Net Technologies.

Este documento es la puerta de entrada a toda la especificación de Shadow-Net Bills, una aplicación para organizar y recordar las obligaciones financieras personales en Colombia. Cada una de las 20 secciones resume en pocos párrafos el contenido del documento fuente y enlaza a él para el detalle completo. Nada aquí duplica los documentos individuales: este maestro orienta y los documentos desarrollan. El lector sin contexto previo entiende el producto, las decisiones y los próximos pasos con solo leer estas páginas, y profundiza siguiendo los enlaces. Todos los enlaces son relativos a este archivo. Las cifras de mercado, precio y conversión se marcan como [HIPÓTESIS] cuando falta validarlas, y los hechos verificados llevan su fuente.

## 1. Visión

Queremos que ninguna persona en Colombia pague una multa por olvidar una factura. Hoy las obligaciones de un hogar están dispersas entre transferencias, correos, aplicaciones de cada proveedor y notas sueltas, y ningún canal muestra el conjunto completo con sus vencimientos. Shadow-Net Bills centraliza todas esas obligaciones en un solo lugar, con fecha de vencimiento, recurrencia y recordatorios, para que el usuario siempre sepa qué debe, cuándo vence y qué ya pagó.

La visión de largo plazo es construir el centro de control de las obligaciones financieras personales en Colombia: primero registro manual con recordatorios (MVP), después importación de facturas con OCR (V1), luego análisis inteligente del gasto (V2) y finalmente integraciones con correo, calendario y finanzas abiertas cuando estén operativas en el país (V3 y futuro). Lo que no cambia con las fases es el principio: Bills organiza y recuerda, no mueve dinero y no pide credenciales bancarias.

Detalle completo en [visión del producto](./product/product-overview.md).

## 2. Problema

El problema central es simple y costoso: las personas olvidan pagar facturas porque sus vencimientos están fragmentados en muchos canales. Cada proveedor comunica por un medio distinto y ninguno muestra el total del mes. Las recurrencias se mezclan (mensuales, trimestrales, anuales y únicas), las billeteras permiten pagar facturas sueltas pero no consolidan obligaciones entre proveedores [DATO] (Fuente: nequi.com.co, consultado 2026-10-07), y un día de mora genera intereses y recargos que rondan los $20.000 COP por evento [HIPÓTESIS] (Fuente: investigación interna Shadow-Net, 2026-10-07).

El contexto colombiano agrava el cuadro: no existe una banca abierta estable que agregue cuentas de forma confiable [DATO] (Fuente: minhacienda.gov.co, consultado 2026-10-07), los rieles dominantes son PSE y Bre-B para mover dinero (no para recordar), y Nequi reporta entre 27 y 29 millones de usuarios sin ofrecer seguimiento entre proveedores [DATO] (Fuente: nequi.com.co, consultado 2026-10-07). La carga mental de revisar cuenta por cuenta recae sobre una sola persona del hogar [HIPÓTESIS], y ese es el dolor que Bills ataca.

Detalle completo en [visión del producto](./product/product-overview.md) y [plan de validación](./product/product-validation.md).

## 3. Solución

Shadow-Net Bills es una aplicación web donde el usuario registra cada obligación financiera (nombre, monto en COP, categoría, fecha de vencimiento, frecuencia y método de pago genérico), la ve en un panel y un calendario mensual, la marca como pagada y recibe recordatorios por correo electrónico 3, 1 y 0 días antes del vencimiento. El flujo de valor es directo: registrar una vez, ver el mes completo, recibir el aviso a tiempo y guardar el historial de pagos.

La solución es deliberadamente manual-primero: el usuario registra sus obligaciones a mano durante el MVP de 8 semanas, sin integraciones bancarias, sin iniciación de pagos y sin IA ni OCR. La importación automática llega en V1, el análisis inteligente en V2 y las integraciones externas en V3. Esta secuencia permite validar si el dolor es real antes de invertir en tecnología costosa, con solo 6 tablas y una API pequeña como base.

Detalle completo en [visión del producto](./product/product-overview.md).

## 4. Público objetivo

El público objetivo son personas en Colombia, entre 22 y 55 años, que gestionan 5 o más obligaciones mensuales y hoy dependen de recordatorios dispersos como correos, notas y memoria [HIPÓTESIS] (Fuente: investigación interna Shadow-Net, 2026-10-07). Los segmentos prioritarios son tres: hogares que pagan arriendo y servicios y olvidan el agua o el gas cuando el correo se pierde; trabajadores con tarjetas y suscripciones que pagan intereses por un día de retraso; y madres, padres y cuidadores que centralizan los pagos del hogar (educación, seguros, impuestos) y cargan con la carga mental.

Tres personas concretas representan al público: Ana (34 años, empleada en Bogotá que centraliza su hogar, 11 obligaciones), Carlos (27 años, desarrollador en Medellín con tarjetas y suscripciones, 9 obligaciones) y María (45 años, comerciante en Cali que mezcla hogar y negocio, 13 obligaciones). Quedan fuera del público inicial las empresas que necesitan facturación electrónica, quienes buscan un agregador bancario automático y quienes solo quieren pagar una factura suelta sin organizar nada.

Detalle completo en [visión del producto](./product/product-overview.md) y [plan de validación](./product/product-validation.md).

## 5. Propuesta de valor

La propuesta de valor seleccionada es **Nunca más olvides pagar una factura**. Se evaluaron cuatro alternativas con cinco criterios (claridad, diferenciación, valor percibido, facilidad de comunicar y ajuste al mercado colombiano): la opción A ganó con 46/50 frente a "Tus cuentas, bajo control" (30/50), "Saber cuánto debes, antes de que te lo cobren" (36/50) y "El control total de tus pagos recurrentes" (30/50). Los puntajes son juicio del equipo [HIPÓTESIS]. La frase ganadora describe el resultado exacto, cabe en un titular, se recuerda tras una lectura y nombra el dolor local: la dispersión de canales que PSE y billeteras no resuelven porque resuelven el pago pero no la memoria [HIPÓTESIS].

El diferenciador es una combinación, no una función aislada: seguimiento de obligaciones con recurrencia real, más calendario mensual consolidado, más contexto colombiano nativo, más recordatorios por correo en 3, 1 y 0 días, más análisis futuro del gasto. Ningún competidor local reúne esos cinco elementos a precio accesible [HIPÓTESIS]. Por eso Bills no se describe como una app para recordar pagos, frase que lo pondría a competir con la alarma del celular.

Detalle completo en [visión del producto](./product/product-overview.md) y [análisis competitivo](./product/competitive-analysis.md).

## 6. Alcance (MVP/V1/V2/Futuro)

El alcance se ordena por fases con la escala MoSCoW. El MVP (MUST HAVE, 8 semanas, manual-primero) incluye autenticación con correo y contraseña, CRUD de obligaciones con las 9 categorías fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros), panel con tarjetas de resumen, calendario mensual, marcar como pagada con historial, recordatorios por correo en offsets de 3, 1 y 0 días, resumen mensual, configuración, exportación CSV y borrado de cuenta. Fuera del MVP (WON'T HAVE) quedan IA y OCR, suscripciones y cuotas como módulos propios, analítica, push, Google OAuth, MFA, modo oscuro, lectura de correo, sincronización de calendarios, iniciación de pagos, banca abierta y app nativa.

V1 (SHOULD HAVE) trae OCR e importación: parseo gratuito de XML DIAN, digitalización con OCR en la nube y módulos de suscripciones y cuotas. V2 (COULD HAVE) trae análisis inteligente: detección determinista de cambios de precio y resumen financiero asistido opcional. V3 y futuro cubren automatización e integraciones (correo, calendario) y banca abierta cuando sea operativa en Colombia, con uso realista esperado desde 2027.

Detalle completo en [requisitos](./product/requirements.md) y [roadmap](./product/roadmap.md).

## 7. Requerimientos

Los requerimientos se numeran RF-001 a RF-025 para el MVP (todos MUST HAVE) y RNF-001 a RNF-016 para las cualidades transversales, con futuros RF-V1 y RF-V2 por fase. Cubren autenticación y cuenta (registro, login con bloqueo progresivo, logout, recuperación), CRUD de obligaciones sobre la tabla `obligations` con columna `next_due_date`, las 9 categorías fijas, panel y calendario, marcar como pagada con generación automática del siguiente vencimiento, historial, recordatorios por correo en 3, 1 y 0 días, resumen mensual, configuración, exportación CSV y borrado de cuenta. Los no funcionales exigen paso por API con RLS, hash de contraseñas, TLS, carga inicial en menos de 2 segundos, disponibilidad de 99,5 por ciento, cumplimiento de la Ley 1581 de 2012, accesibilidad WCAG 2.1 AA, diseño responsive mobile-first, envío idempotente y respaldo diario.

La matriz de trazabilidad vincula cada requisito con su caso de uso y su fase. Nada autoriza mover dinero, guardar credenciales bancarias ni leer portales de banca, y ningún MUST pertenece a una fase posterior.

Detalle completo en [requisitos](./product/requirements.md).

## 8. Casos de uso

Los 16 casos de uso describen el comportamiento del sistema con formato Actor, Precondiciones, Flujo principal, Excepciones y Postcondiciones. Los 12 del MVP cubren registrar, editar y eliminar obligaciones (con confirmación), marcar como pagada, generar el siguiente vencimiento por recurrencia, configurar y recibir recordatorios, consultar el calendario, ver el resumen mensual, exportar a CSV, eliminar la cuenta y recuperar la contraseña. Los 4 de fases posteriores se marcan con su fase: suscripciones y cuotas se registran en el MVP como obligaciones recurrentes, mientras la importación de facturas es V1 y la revisión de análisis es V2.

Los actores son el usuario registrado, el visitante y el sistema programador de recordatorios. Cada caso de pagos deja claro que marcar como pagada es un registro manual que no ejecuta ninguna transacción externa.

Detalle completo en [casos de uso](./product/use-cases.md).

## 9. Arquitectura

La arquitectura del MVP es un monolito modular en dos piezas: un frontend estático en Cloudflare Pages (React con TypeScript y Vite) y una API única en FastAPI sobre Render o Fly que concentra la lógica, el programador de recordatorios APScheduler embebido y el acceso a datos. La persistencia y la autenticación las provee Supabase (PostgreSQL con RLS más Auth con JWT) y el envío de correos usa AWS SES. El tráfico entra por CDN, la API valida el JWT en cada petición y el programador despierta cada hora para encolar los avisos de 3, 1 y 0 días con registro idempotente en `notification_log`.

Las decisiones justificadas son monolito frente a microservicios, Supabase frente a AWS propio, cliente Supabase único sin SQLAlchemy, APScheduler embebido frente a Celery, y exclusión de Kubernetes, Kafka, Redis y service mesh en el MVP. La regla de acceso es estricta: el frontend nunca toca la base directo, toda operación pasa por la API, la clave `service_role` nunca se usa para lecturas con alcance de usuario y RLS jamás se omite. Para V1 y siguientes solo se definen interfaces abstractas (DocumentProcessor, NotificationService, AnalyticsService), sin implementarlas.

Detalle completo en [arquitectura](./architecture/architecture.md).

## 10. Stack tecnológico

El stack aprobado combina React con TypeScript y Vite para el frontend estático, FastAPI con Python y Pydantic para la API monolítica, Supabase (PostgreSQL más Auth más Storage más RLS nativo, plan Pro de $25 al mes) para datos e identidad, cliente Supabase como única vía de acceso sin SQLAlchemy, APScheduler embebido para los recordatorios, AWS SES (cerca de $0,10 por cada 1.000 correos) para el envío, y Render o Fly.io para la API más Cloudflare Pages sin costo para el frontend. Cada componente documenta su alternativa evaluada (Next.js, Django, Auth0 con PostgreSQL propio, SQLAlchemy, Celery, SendGrid), su razón y su costo.

Los criterios de selección fueron simplicidad operativa para construir en 8 semanas, costo total entre $30 y $55 al mes para 100 a 500 usuarios, encaje con el dominio de recordatorios, seguridad por defecto y camino de evolución hacia V1 y V2 sin reescribir la base. La IA y el OCR quedan preparados pero excluidos del MVP.

Detalle completo en [selección de stack](./technical/stack-selection.md) e [infraestructura y costos](./technical/infrastructure.md).

## 11. Modelo de datos

El modelo del MVP tiene exactamente 6 tablas: profiles (perfil 1 a 1 por usuario), categories (filas de sistema con las 9 categorías fijas más filas propias de cada usuario), obligations (la tabla central con nombre, monto decimal, moneda COP, frecuencia, vencimiento, columna `next_due_date`, método de pago genérico, notas, recordatorios y estado), payments (historial de pagos), notification_log (registro de envíos con clave única por obligación, tipo y fecha para idempotencia) y app_settings (ajustes 1 a 1 por usuario). Las convenciones fijan zona `America/Bogota`, borrado en cascada con `ON DELETE CASCADE`, montos no negativos y claves primarias UUID.

Cada tabla lleva índices sobre `user_id`, fechas de vencimiento y estados, y políticas RLS en SQL con `auth.uid() = user_id`, con lectura de filas de sistema permitida en categories. Las tablas previstas para V1 y siguientes (subscriptions, installments, documents, extracted_data, audit_logs) se listan solo como roadmap, sin diseño, y el modelo nunca guarda contraseñas ni datos de tarjetas.

Detalle completo en [modelo de datos](./architecture/database.md).

## 12. Seguridad

La seguridad se rige por mínimo privilegio, defensa en profundidad, minimización de datos, fallo seguro y trazabilidad sin datos sensibles en registros. El modelo de amenazas cubre 16 filas con su fase (MVP, V1 o futuro): robo de sesión, IDOR y acceso a datos de otro usuario, abuso de API, fuerza bruta, almacenamiento inseguro, gestión de secretos, logs sensibles, enumeración de cuentas, suplantación por recordatorios y borrado incompleto en el MVP; subidas maliciosas, prompt injection y fuga hacia proveedores de IA en V1; webhooks falsificados y OAuth mal configurado en futuro. Cada amenaza lleva vector, impacto y mitigación.

Los controles del MVP son RLS en las 6 tablas con `auth.uid() = user_id`, JWT de corta duración con rotación, límite de 100 req/min por usuario, validación de entradas en servidor, TLS 1.3 con cifrado en reposo, secretos solo en variables de entorno y remitente autenticado con SPF, DKIM y DMARC. La respuesta a incidentes sigue detección, contención, notificación a la SIC dentro de los 15 días hábiles y post-mortem, y el checklist pre-lanzamiento exige más de 10 verificaciones. No se afirma ninguna certificación inexistente.

Detalle completo en [seguridad](./architecture/security.md) y [marco legal](./legal/regulatory-compliance.md).

## 13. IA

La IA no forma parte del MVP por decisión explícita: todo lo determinista (CRUD, recurrencias, recordatorios, resúmenes por agregación, detección de cambios de precio) se resuelve sin modelos. La estrategia rige V1 y posterior con un principio rector claro: IA solo donde aporta valor medible y no existe alternativa determinista más barata, rápida y auditable. La matriz evalúa seis casos y concluye que el único que requiere IA en V1 es la extracción de facturas desde imagen o PDF sin XML; clasificación, anomalías, cambios de precio y resúmenes se resuelven con reglas, SQL o estadística simple.

El pipeline OCR es escalonado para minimizar costo: primero parseo local gratuito del XML DIAN (UBL 2.1 con CUFE, que cubriría 40 a 60 por ciento de las facturas), luego OCR en la nube a unos $10 por cada 1.000 páginas, y solo con confianza menor a 0,85 se escala a un LLM pequeño (menos de $0,01 por factura), siempre con confirmación humana obligatoria. La privacidad exige consentimiento explícito de transferencia internacional y acuerdos de tratamiento con cada proveedor, y la arquitectura desacoplada tras la interfaz DocumentProcessor permite cambiar de proveedor sin reescribir el producto.

Detalle completo en [estrategia de IA](./architecture/ai-strategy.md).

## 14. UX

La experiencia se guía por ocho principios: registro en menos de un minuto, claridad permanente sobre el próximo vencimiento ordenado por `next_due_date`, recordatorios predecibles en offsets fijos de 3, 1 y 0 días, categorías fijas en español colombiano, lenguaje honesto que nunca promete mover dinero, estados vacíos que enseñan, móvil primero y errores accionables con carga visible. Los cinco flujos principales son registro con onboarding y primera obligación en 24 horas, marcar como pagada con recurrencia automática, configurar recordatorios, consultar el calendario y exportar datos.

Las 15 pantallas se especifican con propósito, información, acciones, estados vacíos, errores, loading y comportamiento responsive: 11 del MVP (Login, Onboarding, Dashboard, Obligaciones, Detalle, Crear y Editar, Calendario, Resumen mensual, Notificaciones, Configuración y Perfil) y 4 conceptuales de fases posteriores (Suscripciones y Cuotas en V1, Documentos en V1, Análisis en V2). Los montos usan formato $20.000 y el método de pago registrado es solo una nota informativa.

Detalle completo en [flujos de usuario](./ux/user-flows.md).

## 15. MVP (alcance exacto y criterios de éxito)

El MVP es una aplicación usable en 8 semanas donde un usuario real crea su primera obligación en su primer día sin ayuda. El alcance exacto es: autenticación con correo y contraseña; CRUD de obligaciones con nombre, monto en COP, categoría de las 9 fijas, vencimiento, frecuencia (única, semanal, mensual, trimestral o anual), método genérico (tarjeta, transferencia, efectivo, débito automático, otro) y notas; panel con tarjetas de resumen; calendario mensual con estados; marcar como pagada con historial y recurrencia automática; recordatorios por correo en 3, 1 y 0 días; resumen mensual con comparativo simple; configuración con zona `America/Bogota`; exportación CSV; y borrado de cuenta. Todo lo demás (OCR, módulos propios de suscripciones y cuotas, analítica, push, OAuth, MFA, integraciones) queda fuera.

Los criterios de éxito son hipótesis por validar [HIPÓTESIS]: activación del 70 por ciento con primera obligación en 24 horas, retención del 40 por ciento a 30 días, promedio de 8 o más obligaciones por usuario, 60 por ciento o más en "muy decepcionado" de Sean Ellis y NPS mayor a 30. La regla de decisión es GO si se alcanzan, PIVOT si hay señales parciales, y STOP si la activación cae bajo 50 por ciento o más del 60 por ciento dice que no le importaría el cierre.

Detalle completo en [requisitos](./product/requirements.md) y [plan de validación](./product/product-validation.md).

## 16. Roadmap (fases y compuertas)

El roadmap ordena el trabajo en 8 fases que se desbloquean por métricas, no por tiempo: Fase 0 (validación del problema, 20 entrevistas y 100 encuestas como compuerta de entrada), Fase 1 (fundaciones técnicas y de diseño), Fase 2 (MVP de 8 semanas), Fase 3 (validación con usuarios reales contra los umbrales GO, PIVOT y STOP), Fase 4 (OCR e importación), Fase 5 (inteligencia), Fase 6 (automatización e integraciones) y Fase 7 (escalamiento). Cada fase define objetivo, entregables, criterios de salida medibles, dependencias y riesgos, y las duraciones son solo [ESTIMACIÓN] sin fechas calendario comprometidas.

Las compuertas GO, PIVOT y STOP reutilizan los umbrales del producto (activación 70 por ciento, retención 40 por ciento a 30 días, 8 obligaciones promedio, 60 por ciento Sean Ellis, NPS mayor a 30). La sección de exclusiones explica qué queda fuera de cada fase y por qué, con el OCR explícitamente fuera del MVP.

Detalle completo en [roadmap](./product/roadmap.md).

## 17. Modelo de negocio

El modelo es suscripción simple con principios no negociables: el usuario paga por el producto y nunca con sus datos, sin venta ni alquiler de datos hoy ni en versiones futuras, y con precios accesibles para Colombia [DECISIÓN]. El tier gratuito cubre hasta 15 obligaciones con recordatorios, calendario, resumen básico y exportación [HIPÓTESIS]; el tier Premium ofrece obligaciones ilimitadas, OCR en V1, análisis en V2 y soporte prioritario, con techo propuesto de hasta $20.000 COP al mes o alrededor de $80.000 COP al año en plan anual [HIPÓTESIS] (Fuente: investigación interna Shadow-Net, 2026-10-07).

Las anclas de disposición a pagar son Finku a $20.000 al mes [DATO] (Fuente: finku.co, consultado 2026-10-07), Spotify a $18.900 [HIPÓTESIS], Netflix cerca de $50.000 [HIPÓTESIS] y la multa típica de $20.000 por olvido [HIPÓTESIS]: un solo recargo evitado al mes pagaría la suscripción, y esa es la tesis central de valor por comprobar. Los costos son infraestructura de $30 a $55 al mes para el MVP e IA cercana a $0,10 por usuario al mes a escala, con unit economics (conversión 5 a 8 por ciento, CAC y LTV) marcados como [HIPÓTESIS] con metodología explícita. Nada de precios finales se publica sin validar con usuarios.

Detalle completo en [modelo de negocio](./business/business-model.md) y [análisis de mercado](./business/market-analysis.md).

## 18. Riesgos

Los riesgos se agrupan en cinco frentes. En mercado, la doble lectura del espacio vacío: oportunidad real frente a ausencia de demanda, con la tracción pequeña de NoSeMePasa y el dominio de billeteras gratuitas como señales ambiguas que solo las métricas de validación pueden resolver. En competencia, que Nequi o Daviplata añadan recordatorios con su distribución masiva, que PliP extienda su agregación DIAN hacia avisos, o que la banca abierta operativa desde 2027 habilite a agregadores hoy inestables. En modelo, la sensibilidad al precio con techo cercano a $20.000 y la conversión 5 a 8 por ciento aún sin medir [HIPÓTESIS].

En técnica, el sobrediseño prematuro (mitigado al prohibir Kubernetes, microservicios, Kafka y Redis en el MVP), la dependencia de un solo proceso para el programador y la superficie futura de subidas y OCR. En regulación, sanciones de hasta 2.000 SMLMV bajo la Ley 1581, el deber de notificar incidentes a la SIC en 15 días hábiles y las incertidumbres del proyecto de reforma de datos y del cronograma de finanzas abiertas, que se documentan como pendientes y no como ley vigente. Cada riesgo lleva su mitigación en su documento fuente.

Detalle completo en [análisis de mercado](./business/market-analysis.md), [modelo de negocio](./business/business-model.md), [seguridad](./architecture/security.md) y [marco legal](./legal/regulatory-compliance.md).

## 19. Decisiones técnicas

Nueve decisiones clave sostienen el MVP, cada una con su razón. Primera, monolito modular en lugar de microservicios, porque 6 tablas y una veintena de endpoints no justifican la operación distribuida y un equipo pequeño debe lanzar en 8 semanas. Segunda, Supabase (PostgreSQL más Auth más RLS) en lugar de base propia con Auth0, porque Auth, datos y aislamiento por fila conviven en un servicio y cada fila queda protegida por `auth.uid() = user_id` sin sincronizar usuarios. Tercera, cliente Supabase como única vía de acceso y sin SQLAlchemy, porque dos estrategias de acceso duplicarían sesiones, transacciones y puntos de auditoría del aislamiento.

Cuarta, APScheduler embebido en lugar de Celery, porque unos pocos miles de correos diarios no justifican trabajador y cola separados, con job store persistente e instancia única como reglas operativas. Quinta, React con TypeScript y Vite como SPA estática en lugar de Next.js, porque no hay contenido público que indexar y la salida estática encaja con Cloudflare Pages sin servidor Node permanente. Sexta, FastAPI en lugar de Django, porque Pydantic pega los esquemas a la especificación de API sin el andamiaje completo. Séptima, AWS SES en lugar de SendGrid para el correo transaccional por su costo cercano a $0,10 por millar. Octava, Render o Fly.io en us-east-1 en lugar de nube propia, por operación simple y mejor latencia desde Colombia que São Paulo. Novena, OCR y banca abierta excluidos del MVP y preparados solo como interfaces, porque construirlos antes de validar el dolor quemaría presupuesto en tecnología que la demanda aún no reclama.

Detalle completo en [selección de stack](./technical/stack-selection.md) y [arquitectura](./architecture/architecture.md).

## 20. Próximos pasos

La siguiente fase es el desarrollo del MVP tras la aprobación de esta especificación, y arranca por la validación del problema antes de escribir código: 20 entrevistas con el guion de 18 preguntas y la encuesta de 10 preguntas del plan de validación, más el experimento concierge con 20 usuarios gestionados a mano. Solo con evidencia de dolor recurrente y compromiso observable se desbloquean las fundaciones (Fase 1) y la construcción de 8 semanas (Fase 2), según las compuertas del roadmap.

En paralelo a la validación, el equipo debe consultar a un abogado colombiano para confirmar el checklist operativo (política de privacidad, autorizaciones granulares versionadas, contratos de transmisión con proveedores, tabla de retención y playbook de brechas), dado que este paquete es investigación informativa y no asesoramiento jurídico. La estructura de código propuesta (backend, frontend, supabase, automatización y CI) ya está descrita y no debe crearse hasta que la Fase 1 la autorice. Con la aprobación del lector, la especificación queda como fuente oficial y el desarrollo empieza sin volver a decidir lo fundamental.

Detalle completo en [roadmap](./product/roadmap.md), [plan de validación](./product/product-validation.md) y [estructura de repositorio](./technical/repository-structure.md).
