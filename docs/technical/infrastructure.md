# Infraestructura y costos de Shadow-Net Bills

Estado: especificación aprobada para el MVP manual-primero. Los precios citados corresponden a las páginas oficiales de cada proveedor y se verificaron en octubre de 2026. Las proyecciones de costo por escala son [ESTIMACIÓN] con metodología explícita en su sección. Nada de lo descrito en este documento provisiona infraestructura real: es una guía de decisión para cuando inicie la fase de desarrollo.

## 1. Resumen y recomendación

La recomendación para el MVP es un monolito simple desplegado en la nube con servicios gestionados: base de datos, autenticación y almacenamiento en Supabase Pro; API FastAPI en Render o Fly.io; frontend estático en Cloudflare Pages con CDN; correos transaccionales con AWS SES; y monitorización con los niveles gratuitos de los proveedores.

El costo total estimado del stack MVP es de 30-55 USD al mes para un volumen de 100 a 500 usuarios [ESTIMACIÓN] (metodología: suma de los precios base publicados por cada proveedor más un margen por consumo variable de correos y monitoreo, detallada en la sección 3). Esta cifra permite operar el MVP sin sobredimensionar: sin Kubernetes, sin microservicios, sin Kafka, sin Redis y sin service mesh, que quedan explícitamente fuera del MVP por costo y complejidad injustificados.

La decisión central es priorizar servicios gestionados sobre infraestructura propia. El equipo dedica su tiempo a validar el producto (recordatorios que llegan a tiempo, calendario confiable, resumen mensual útil) en lugar de administrar servidores, y el costo mensual se mantiene en un rango que un fundador puede sostener durante la fase de validación.

## 2. Comparativa de proveedores

La siguiente tabla compara los proveedores evaluados para los componentes del MVP. Los precios son los publicados en 2026 e incluyen la fuente y la fecha de consulta. Donde el precio depende del consumo, se indica el rango del nivel relevante para el MVP.

| Proveedor | Componente evaluado | Precio 2026 relevante al MVP | Nivel gratuito | Fuente |
| --- | --- | --- | --- | --- |
| Supabase | Base de datos PostgreSQL, Auth y Storage | Pro 25 USD/mes | 50.000 MAU, 500 MB de base de datos | Fuente: supabase.com/pricing, consultado 2026-10-07 |
| Render | Hospedaje de la API FastAPI | Aproximadamente 25 USD/mes por servicio web estándar | Nivel gratuito con suspensión por inactividad | Fuente: render.com/pricing, consultado 2026-10-07 |
| Fly.io | Hospedaje alternativo de la API FastAPI | Aproximadamente 7-25 USD/mes según tamaño de máquina y tráfico | Asignación gratuita pequeña para pruebas | Fuente: fly.io/pricing, consultado 2026-10-07 |
| DigitalOcean | Alternativa de hospedaje (Droplet o App Platform) | Desde aproximadamente 6-25 USD/mes según plan | Sin nivel gratuito permanente; crédito de bienvenida temporal | Fuente: digitalocean.com/pricing, consultado 2026-10-07 |
| Railway | Alternativa de hospedaje con despliegue simple | Desde aproximadamente 5 USD/mes más consumo | Crédito de prueba limitado para empezar | Fuente: railway.app/pricing, consultado 2026-10-07 |
| AWS | Correos con SES y alternativa de cómputo (EC2 o App Runner) | SES aproximadamente 0,10 USD por cada 1.000 correos | Nivel gratuito de 12 meses en varios servicios | Fuente: aws.amazon.com, consultado 2026-10-07 |
| GCP | Alternativa de cómputo (Cloud Run) y OCR futuro (Document AI) | Cloud Run por consumo desde 0 USD con tráfico bajo; Document AI prebuilt por uso | 1.000 páginas mensuales de Document AI | Fuente: cloud.google.com/pricing, consultado 2026-10-07 |
| Azure | Alternativa de OCR futuro (Document Intelligence) | OCR prebuilt por uso, ver estrategia de IA | 500 páginas mensuales de Document Intelligence | Fuente: azure.microsoft.com/pricing, consultado 2026-10-07 |
| Cloudflare | Páginas estáticas del frontend y CDN | 0 USD en el plan gratuito | Plan gratuito con CDN y Pages incluidos | Fuente: cloudflare.com/plans, consultado 2026-10-07 |

La conclusión de la comparativa es que Supabase Pro es la pieza que más valor aporta por su precio: por $25 al mes se obtiene base de datos con respaldos, autenticación con JWT, almacenamiento de archivos y políticas de acceso por fila en un solo servicio. Para la API, Render ofrece la operación más simple a cambio de un precio fijo cercano a 25 USD al mes, mientras que Fly.io permite empezar más barato (alrededor de 7 USD al mes) a cambio de una configuración algo más manual. DigitalOcean y Railway quedan como alternativas válidas si el equipo ya las conoce, y AWS, GCP y Azure se reservan para servicios puntuales (SES para correo; OCR en la fase V1) en lugar de hospedar todo el sistema.

## 3. Desglose de la recomendación MVP

El rango de 30-55 USD al mes se compone de la siguiente suma, con precios de octubre de 2026:

| Componente | Servicio elegido | Costo mensual | Notas |
| --- | --- | --- | --- |
| Base de datos, Auth y Storage | Supabase Pro | $25 USD/mes | Incluye respaldos automáticos y sin pausa por inactividad. Fuente: supabase.com/pricing, consultado 2026-10-07 |
| API FastAPI | Render estándar o Fly.io | 7-25 USD | Render cerca de 25 USD con operación simple; Fly.io desde 7 USD con máquina pequeña. Fuente: render.com/pricing y fly.io/pricing, consultado 2026-10-07 |
| Frontend y CDN | Cloudflare Pages | 0 USD | Plan gratuito suficiente para el tráfico del MVP. Fuente: cloudflare.com/plans, consultado 2026-10-07 |
| Correo transaccional | AWS SES | Aproximadamente 1 USD | A razón de 0,10 USD por cada 1.000 correos; el volumen del MVP (recordatorios 3, 1 y 0 días antes del vencimiento) se mantiene en pocos miles al mes. Fuente: aws.amazon.com, consultado 2026-10-07 |
| Monitorización y alertas | Niveles gratuitos o plan básico | 0-25 USD | Se empieza en 0 USD con los paneles de cada proveedor y se sube a un plan básico solo si se necesita retención de registros. Ver sección 9 |
| Total MVP | Stack completo | 30-55 USD | Suma de los componentes anteriores con redondeo al alza |

Para el prototipo previo al lanzamiento, el nivel gratuito de Supabase (50.000 MAU y 500 MB de base de datos) permite validar sin pagar la suscripción Pro. El cambio a Pro se hace antes de abrir el servicio a usuarios reales, porque el plan gratuito pausa proyectos inactivos y no ofrece respaldos con la misma garantía. Fuente: supabase.com/pricing, consultado 2026-10-07.

## 4. Proyecciones por escala

Las proyecciones siguientes son [ESTIMACIÓN] (metodología: se parte del costo base del MVP y se escala cada componente según el consumo esperado de base de datos, cómputo, correos y almacenamiento; no son cotizaciones de los proveedores):

| Escala | Usuarios aproximados | Costo mensual estimado | Qué cambia respecto al MVP |
| --- | --- | --- | --- |
| MVP y validación | 100-500 | 30-55 USD | Stack base de la sección 3, sin cambios |
| Crecimiento | 1.000-5.000 | 110-220 USD | Base de datos de mayor capacidad, API con más memoria o segunda instancia, volumen de SES en decenas de miles de correos, monitoreo con retención |
| Escala | 10.000 o más | 350-650 USD | Base de datos dedicada o con réplica de lectura, API con escalado horizontal, CDN con mayor transferencia, respaldos extendidos y monitoreo de pago |

Estas cifras suponen el producto manual-primero sin OCR masivo. El costo de OCR e IA de la fase V1 se presupuesta por separado (aproximadamente 10 USD por cada 1.000 páginas procesadas más el costo del modelo de extracción, con un objetivo menor a 0,01 USD por factura) y se describe en el documento de estrategia de IA. Mantener esa separación evita que el costo de una funcionalidad futura contamine el presupuesto del MVP.

## 5. Estrategias de optimización

Las siguientes estrategias mantienen el costo bajo sin sacrificar confiabilidad, ordenadas de mayor a menor impacto para el MVP:

1. Niveles de almacenamiento por edad del dato. Los comprobantes y archivos adjuntos recientes permanecen en el nivel estándar de Supabase Storage; los archivos de más de un año se mueven a un nivel frío o se comprimen. Esta política se define antes del lanzamiento para que el crecimiento del almacenamiento sea lineal y predecible.
2. Caché de lecturas frecuentes. El resumen mensual y el calendario se calculan una vez por usuario y período y se sirven desde caché con invalidación al crear, editar o pagar una obligación. Esto reduce las consultas repetidas a la tabla `obligations` sin añadir Redis en el MVP: basta la caché HTTP con encabezados de expiración corta y la caché de consultas de React Query en el frontend.
3. Compresión de PDF y de respuestas. Los PDF generados para exportación se comprimen antes de almacenarse; las respuestas JSON de la API usan compresión gzip o brotli; y las imágenes del frontend se sirven en formatos modernos con dimensiones limitadas. Cada medida reduce transferencia y almacenamiento con costo de implementación mínimo.
4. Correos estrictamente necesarios. Los recordatorios se envían solo en los desplazamientos aprobados (3, 1 y 0 días) y solo a usuarios con recordatorios activados. La clave de idempotencia de la tabla `notification_log` evita reenvíos duplicados que costarían dinero en SES y dañarían la confianza del usuario.
5. Cuándo migrar cada pieza. Se migra la API de Fly.io a Render (o a la inversa) solo cuando el costo operativo supera el ahorro durante tres meses consecutivos; se sale de Supabase Pro hacia una base dedicada solo cuando la base supera de forma sostenida el 70 por ciento de los límites del plan; y se contrata monitoreo de pago solo cuando la retención gratuita impide diagnosticar un incidente real. Ninguna migración se hace por anticipación.

## 6. Latencia y región desde Colombia

La región recomendada para todos los servicios es us-east-1 (Norte de Virginia, EE. UU.). El corredor Bogotá a Miami y Norte de Virginia ofrece menor latencia desde Colombia que la región de São Paulo, por la conectividad directa de los cables submarinos del Caribe hacia la costa este de Estados Unidos [ESTIMACIÓN] (metodología: comparación de rutas de cable submarino y mediciones públicas de ida y vuelta; validar con una prueba de latencia propia antes del lanzamiento).

En la práctica esto significa: crear el proyecto de Supabase en la región este de Estados Unidos, desplegar la API en el mismo corredor (Render en Ohio o Virginia, o Fly.io en Miami o Ashburn), y servir el frontend desde la red global de Cloudflare, que enruta automáticamente a cada usuario al borde más cercano. La base de datos y la API deben quedar en la misma región para que la ida y vuelta entre ambas sea mínima; separarlas en continentes distintos es el error más común y el más fácil de evitar.

El objetivo de rendimiento asociado es una carga inicial menor a 2 segundos en conexiones móviles típicas colombianas, con el respaldo de la CDN para los recursos estáticos y consultas paginadas en los listados de obligaciones.

## 7. Créditos para startups

Los programas de crédito para startups pueden cubrir entre 6 y 12 meses de infraestructura si se solicitan antes de necesitarlos. Los más relevantes para este stack son el crédito de inicio de AWS (aplicable a SES y a cómputo futuro), los créditos de Google Cloud (útiles para la fase de OCR con Document AI) y los créditos de Microsoft Azure (alternativa para Document Intelligence). Supabase, Render, Fly.io, DigitalOcean y Railway ofrecen periódicamente créditos o descuentos para proyectos nuevos a través de aceleradoras y programas de fundadores.

La recomendación operativa es solicitar los créditos de AWS desde el inicio aunque el gasto en SES sea mínimo, porque el trámite tarda semanas y el crédito queda disponible para la fase de crecimiento. Los créditos de GCP y Azure se solicitan al acercarse la fase V1 de OCR, no antes, para no consumir su vigencia durante el MVP manual. Todo crédito se registra con su fecha de vencimiento en el seguimiento financiero del proyecto para que su fin no sorprenda al presupuesto.

## 8. Señales para escalar o migrar

Ninguna de las siguientes decisiones se toma por calendario, sino cuando aparece su señal medible:

1. Añadir una segunda instancia de la API cuando la latencia del percentil 95 supere 1 segundo durante una semana completa o cuando el uso de memoria supere el 80 por ciento de forma sostenida.
2. Subir el plan de base de datos cuando el tamaño supere el 70 por ciento del límite del plan Pro o cuando las consultas del calendario y el resumen requieran índices adicionales que el plan actual no sostiene con holgura.
3. Separar el planificador de recordatorios (APScheduler) en un proceso dedicado cuando el envío de correos compita con las peticiones de la API en horas pico.
4. Adoptar una CDN de pago o un plan superior de Cloudflare cuando la transferencia mensual supere de forma sostenida el nivel gratuito.
5. Evaluar caché dedicada o réplica de lectura cuando el número de usuarios activos supere 5.000 y las proyecciones de la sección 4 indiquen que el costo ya entró en el rango de 110-220 USD.

Migrar antes de ver estas señales desperdicia dinero y atención; migrar después de verlas durante más de un mes pone en riesgo la experiencia del usuario. El seguimiento mensual del costo real frente a las proyecciones es la práctica que mantiene este equilibrio.

## 9. Monitorización

La monitorización del MVP usa los niveles gratuitos de cada proveedor y se amplía solo ante necesidad probada. La configuración mínima antes del lanzamiento incluye: panel de salud de la API (tiempo de respuesta, tasa de errores 5xx y uso de memoria) en Render o Fly.io; panel de base de datos (conexiones, tamaño y consultas lentas) en Supabase; tasa de entrega y rebotes de correo en AWS SES; y disponibilidad del frontend en Cloudflare.

Las alertas mínimas son tres: caída de la API (sin respuestas exitosas durante 5 minutos), tasa de errores superior al 1 por ciento durante 15 minutos, y cola de recordatorios sin procesar pasada su hora programada. Cada alerta llega al canal que el equipo revise a diario, con un responsable nombrado y un procedimiento escrito de dos o tres pasos. El monitoreo de pago con retención extendida de registros se contrata cuando un incidente real no pueda diagnosticarse con la retención gratuita; mientras tanto, el presupuesto de monitoreo permanece en 0 USD.

## 10. Copias de seguridad

La política de respaldos del MVP tiene tres capas. La primera es el respaldo automático diario de Supabase Pro, con retención incluida en el plan de $25 y restauración probada al menos una vez antes del lanzamiento. La segunda es la exportación semanal cifrada de las tablas críticas (`obligations`, `payments`, `profiles`) a un almacenamiento externo en la misma región us-east-1, de modo que exista una copia fuera del proveedor principal. La tercera es el control de versiones de la configuración: variables de entorno documentadas, migraciones SQL versionadas y procedimientos de restauración escritos paso a paso.

La prueba de restauración es obligatoria y se repite cada trimestre: un respaldo que nunca se restauró no es un respaldo, es una esperanza. El borrado de cuenta de un usuario elimina sus datos personales según la política de retención documentada en el análisis regulatorio, sin afectar los respaldos cifrados cuya rotación sigue su propio calendario. El costo de esta política está incluido en el rango MVP de 30-55 USD y no requiere servicios adicionales.
