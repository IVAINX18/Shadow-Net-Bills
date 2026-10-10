# Roadmap de Shadow-Net Bills: fases con compuertas de validación

Shadow-Net Bills es una aplicación para organizar y recordar las obligaciones financieras personales en Colombia. Este roadmap define el orden de construcción por fases, donde cada fase solo se desbloquea cuando se cumplen sus criterios de salida medibles. Nada avanza por calendario: avanza por evidencia.

## Principio rector

Las fases se desbloquean por métricas, no por tiempo. Cada fase termina en una compuerta de validación con criterios de salida medibles. Solo cuando la compuerta se cumple, la fase siguiente queda desbloqueada.

Las duraciones mencionadas en este documento son orientativas y se marcan siempre como [ESTIMACIÓN]. No son compromisos ni fechas prometidas. Sirven únicamente para dimensionar el esfuerzo y planificar la capacidad del equipo.

Reglas del principio rector:

- Ninguna fase se considera terminada sin cumplir sus criterios de salida.
- La decisión de avanzar, ajustar o detener se toma en la compuerta con datos reales.
- El MVP es manual-primero: sin OCR, sin inteligencia artificial, sin integraciones bancarias y sin movimiento de dinero.
- La propuesta de valor que guía todo el roadmap es "Nunca más olvides pagar una factura" [DECISIÓN].
- Los umbrales de validación de este roadmap son los mismos definidos en la estrategia de producto [DECISIÓN].

## Fase 0 — Validación del problema

Duración [ESTIMACIÓN]: 2 semanas de trabajo de descubrimiento, sin código de producto.

El objetivo de esta fase es confirmar que el problema es real, frecuente y doloroso antes de construir nada. El objetivo se cumple cuando existe evidencia de que personas con múltiples obligaciones mensuales olvidan pagos y pagan multas evitables.

Entregables de la fase:

- Guion de entrevistas con 18 preguntas en 5 secciones (contexto, descubrimiento del dolor, soluciones actuales, prueba de solución y compromiso).
- Encuesta de 10 preguntas con opciones cerradas sobre número de obligaciones, olvidos, multas pagadas, método actual, satisfacción, estrés, utilidad percibida, disposición a pagar y funcionalidades más valoradas.
- Registro de entrevistas realizadas con hallazgos por patrón (frecuencia de olvidos, severidad, soluciones actuales).
- Documento de hipótesis del problema con intensidad del dolor y disposición a pagar marcadas como [HIPÓTESIS].

Los criterios de salida de esta fase son la compuerta de entrada al desarrollo:

- Al menos 20 entrevistas realizadas a personas del público objetivo.
- Al menos 100 respuestas de encuesta con patrones analizables.
- Evidencia de que el olvido de pagos es recurrente (frecuencia y severidad documentadas).
- Al menos 3 de cada 10 entrevistados muestran compromiso concreto (dejar correo para la beta o aceptar prueba manual) [HIPÓTESIS].

Dependencias de la fase: ninguna. Es la primera fase y solo requiere acceso al público objetivo y a los instrumentos de investigación.

Riesgos de la fase:

- Sesgo de cortesía: los entrevistados dicen que les gusta la idea pero no se comprometen. Mitigación: pedir compromiso observable (correo, tiempo para prueba manual).
- Muestra no representativa: entrevistar solo a personas cercanas con perfil digital alto. Mitigación: incluir perfiles con distinto nivel de bancarización y organización.
- Enamorarse de la solución antes de entender el problema. Mitigación: las primeras entrevistas no muestran pantallas ni describen la aplicación.

## Fase 1 — Fundaciones

Duración [ESTIMACIÓN]: 2 semanas de preparación técnica y de diseño, sin funcionalidad visible para el usuario final.

El objetivo de esta fase es dejar lista la base para construir el MVP sin improvisar después. El objetivo se cumple cuando la arquitectura, el modelo de datos, el diseño de pantallas y la infraestructura están definidos y revisados.

Entregables de la fase:

- Arquitectura de monolito definida: aplicación web, API, base de datos gestionada, programador de recordatorios embebido y servicio de correo.
- Modelo de datos MVP de 6 tablas (profiles, categories, obligations con columna next_due_date, payments, notification_log, app_settings) con políticas de acceso por usuario.
- Especificación de API con base /api/v1/obligations y convenciones de autenticación, errores y límites.
- Diseño de las 11 pantallas del MVP con estados vacíos, errores y comportamiento móvil.
- Entorno de desarrollo y producción configurado con costo mensual estimado entre $30 y $55 [ESTIMACIÓN].

Los criterios de salida de esta fase son la compuerta técnica:

- Los 6 documentos de arquitectura y UX están completos y sin marcadores pendientes.
- El modelo de datos cubre las 9 categorías fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros) y los offsets de recordatorio {3,1,0} días.
- La revisión de seguridad previa confirma que no se almacenan credenciales bancarias ni números completos de tarjeta, y que toda lectura pasa por la API.
- El despliegue de prueba responde correctamente en los entornos de desarrollo y producción.

Dependencias de la fase: resultados de la Fase 0 (el problema validado justifica invertir en fundaciones).

Riesgos de la fase:

- Sobrediseño: preparar infraestructura para una escala que aún no existe. Mitigación: prohibir en MVP Kubernetes, microservicios, Kafka, Redis y mallas de servicios.
- Acoplamiento prematuro con proveedores de IA u OCR. Mitigación: el MVP no incluye OCR; las interfaces de V1 solo se definen, no se implementan.
- Decisiones de pila revertidas a mitad del MVP. Mitigación: la selección de pila queda registrada como [DECISIÓN] antes de escribir código.

## Fase 2 — MVP

Duración [ESTIMACIÓN]: 8 semanas de construcción del producto mínimo viable manual-primero.

El objetivo de esta fase es entregar una aplicación usable que permita registrar obligaciones, recordar vencimientos por correo y consultar el calendario mensual. El objetivo se cumple cuando un usuario real puede crear su primera obligación en su primer día de uso sin ayuda.

Entregables de la fase:

- Autenticación con correo y contraseña, recuperación de contraseña, configuración y borrado de cuenta con exportación de datos.
- Gestión completa de obligaciones (crear, editar, eliminar con confirmación) con nombre, monto en COP, categoría fija, fecha de vencimiento, frecuencia (one_time, weekly, monthly, quarterly, yearly), método de pago genérico (tarjeta, transferencia, efectivo, débito automático, otro) y notas.
- Generación del siguiente vencimiento por recurrencia usando la columna next_due_date de la tabla obligations.
- Tablero con tarjetas de resumen, calendario mensual, historial de pagos y resumen mensual.
- Recordatorios por correo a 3, 1 y 0 días del vencimiento con envío idempotente.
- Exportación de datos en CSV.

Los criterios de salida de esta fase son la compuerta de calidad interna:

- Los 25 o más requisitos funcionales del MVP están implementados y verificados.
- La lista de verificación de seguridad previa al lanzamiento está completa (al menos 10 ítems).
- El tiempo de carga de las pantallas principales es inferior a 2 segundos en condiciones normales [ESTIMACIÓN].
- Cero defectos críticos abiertos y cero rutas que expongan datos de otro usuario.
- El OCR y la inteligencia artificial NO forman parte del MVP: cualquier rastro de OCR en esta fase es un defecto de alcance.

Dependencias de la fase: fundaciones de la Fase 1 (arquitectura, datos, API, diseño, entornos).

Riesgos de la fase:

- Filtración de alcance: añadir modo oscuro, OAuth, MFA, notificaciones push o analíticas. Mitigación: la lista de fuera del MVP es explícita y cualquier añadido requiere moverlo a su fase.
- Recordatorios no enviados o duplicados. Mitigación: programador con instancia única y clave de idempotencia en notification_log.
- Complejidad de recurrencia (cuotas, frecuencias mixtas). Mitigación: en MVP las suscripciones se registran como obligación recurrente de categoría Suscripciones y las cuotas como obligación de categoría Financiero con notas; los módulos específicos llegan en V1.

## Fase 3 — Validación con usuarios reales

Duración [ESTIMACIÓN]: 4 semanas de operación del MVP con usuarios reales y medición continua.

El objetivo de esta fase es comprobar con datos que el MVP retiene y genera el hábito esperado. El objetivo se cumple cuando las métricas de activación, retención, densidad de uso y satisfacción alcanzan los umbrales definidos o cuando los datos obligan a ajustar o detener.

Entregables de la fase:

- Cohorte de al menos 100 usuarios activos usando el MVP en condiciones reales.
- Tablero de métricas con activación, retención a 30 días, promedio de obligaciones por usuario, prueba de Sean Ellis y NPS.
- Informe de decisión GO, PIVOT o STOP con datos y motivos.
- Lista priorizada de ajustes si la decisión es ajustar (PIVOT).

Los criterios de salida de esta fase son la compuerta de validación principal:

- Activación: al menos 70% de los registrados crean su primera obligación dentro de las primeras 24 horas [HIPÓTESIS].
- Retención: al menos 40% de los usuarios siguen activos al día 30 [HIPÓTESIS].
- Densidad: promedio de 8 o más obligaciones por usuario [HIPÓTESIS].
- Satisfacción: al menos 60% responde "muy decepcionado" si dejara de existir el producto (prueba de Sean Ellis) y NPS superior a 30 [HIPÓTESIS].
- Si las métricas quedan por debajo pero cerca, la salida es PIVOT con ajustes acotados. Si la activación es inferior al 50% o más del 60% responde "nada decepcionado", la salida es STOP.

Dependencias de la fase: MVP desplegado y estable de la Fase 2, más instrumentos de medición y contacto con usuarios.

Riesgos de la fase:

- Vanidad de métricas: celebrar registros sin activación real. Mitigación: la activación se mide por primera obligación creada, no por cuentas creadas.
- Muestra pequeña o sesgada que invalida los porcentajes. Mitigación: no decidir con menos de 100 usuarios y segmentar por perfil.
- Ignorar la señal de STOP por apego a lo construido. Mitigación: los umbrales de STOP son vinculantes y están escritos antes de medir.

## Fase 4 — OCR e importación

Duración [ESTIMACIÓN]: 6 semanas de construcción, solo si la Fase 3 decidió GO.

El objetivo de esta fase es reducir la carga manual permitiendo importar facturas por foto, PDF o XML de facturación electrónica. El objetivo se cumple cuando un usuario puede importar una factura y solo necesita confirmar los datos extraídos.

Entregables de la fase:

- Importación de XML de facturación electrónica con parseo gratuito (ruta preferente para facturas electrónicas colombianas).
- OCR en la nube para fotos y PDF con pipeline escalonado: XML gratis, luego OCR (costo aproximado de $10 por cada 1.000 páginas [ESTIMACIÓN]), luego modelo de lenguaje solo en confianza baja (umbral 0,85) y confirmación humana siempre obligatoria.
- Módulo de suscripciones y módulo de cuotas como entidades propias (ya no solo notas en obligaciones).
- Bóveda de documentos con URLs firmadas y validación de tipo de archivo.
- Consentimiento explícito de transferencia internacional de datos y acuerdos de tratamiento con proveedores.

Los criterios de salida de esta fase son la compuerta de extracción fiable:

- Al menos 80% de las importaciones de XML quedan correctas sin corrección manual [HIPÓTESIS].
- Al menos 70% de las importaciones por OCR quedan correctas con solo confirmación [HIPÓTESIS].
- Costo de extracción por factura inferior a $0,01 en promedio gracias al pipeline escalonado [ESTIMACIÓN].
- Cero documentos accesibles sin autenticación y cero archivos almacenados sin validación.

Dependencias de la fase: decisión GO de la Fase 3, más proveedor de OCR contratado y diseño de pantallas de Documentos, Suscripciones y Cuotas.

Riesgos de la fase:

- Costos de IA descontrolados por procesar todo con el modelo más caro. Mitigación: caché por identificador de factura, procesamiento por lotes y ruta gratuita primero.
- Fuga de datos a proveedores externos. Mitigación: minimización de datos, consentimiento y borrado según política.
- Expectativa de perfección del OCR. Mitigación: la confirmación humana es parte del flujo, no un parche.

## Fase 5 — Inteligencia

Duración [ESTIMACIÓN]: 6 semanas de construcción, solo si la Fase 4 mantiene retención y costos bajo control.

El objetivo de esta fase es convertir los datos acumulados en información útil sin prometer asesoría financiera. El objetivo se cumple cuando el usuario entiende sus patrones de gasto y anticipa cambios con reglas transparentes.

Entregables de la fase:

- Detección de cambios de precio por comparación determinista entre periodos.
- Detección de anomalías con estadística simple (desviaciones frente al historial del usuario).
- Resumen financiero del mes con modelo de lenguaje opcional y textos revisables.
- Pantalla de Análisis con explicaciones de cada hallazgo (qué cambió, desde cuándo, por qué se muestra).
- Clasificación automática por reglas (sin IA) como base; la IA solo se usa donde aporta valor diferencial.

Los criterios de salida de esta fase son la compuerta de utilidad percibida:

- Al menos 50% de los usuarios activos consulta la pantalla de Análisis cada mes [HIPÓTESIS].
- Al menos 70% de los hallazgos mostrados son marcados como útiles o correctos [HIPÓTESIS].
- Cero recomendaciones presentadas como asesoría financiera garantizada.
- El costo de IA por usuario se mantiene alrededor de $0,10 al mes a escala [ESTIMACIÓN].

Dependencias de la fase: volumen de datos de las Fases 2 a 4 (sin historial no hay patrones) y definición de textos y límites de responsabilidad.

Riesgos de la fase:

- Alucinaciones presentadas como hechos financieros. Mitigación: todo hallazgo cita sus datos origen y admite corrección.
- Funcionalidad vistosa pero no usada. Mitigación: la compuerta exige uso mensual real, no clics de curiosidad.
- Complejidad de modelos propios. Mitigación: estadística simple y reglas primero; modelos a medida quedan fuera.

## Fase 6 — Automatización e integraciones

Duración [ESTIMACIÓN]: 8 semanas de construcción, solo si la inteligencia de la Fase 5 demuestra uso recurrente.

El objetivo de esta fase es reducir aún más el trabajo manual conectando el correo, el calendario y canales de aviso adicionales. El objetivo se cumple cuando una factura que llega al correo puede convertirse en obligación con una confirmación.

Entregables de la fase:

- Lectura de facturas recibidas por correo con consentimiento explícito y alcance mínimo de permisos.
- Sincronización de vencimientos con calendarios externos (solo lectura y creación de eventos, sin borrados destructivos).
- Notificaciones push y autenticación con Google como opción.
- Autenticación de doble factor opcional.
- Centro de integraciones con conexión, desconexión y auditoría de accesos.

Los criterios de salida de esta fase son la compuerta de confianza:

- Al menos 30% de los usuarios activos conecta al menos una integración [HIPÓTESIS].
- Tasa de conversión de correo a obligación confirmada de al menos 50% [HIPÓTESIS].
- Cero incidentes de acceso excesivo a correos o calendarios en auditoría.
- La desconexión de cualquier integración elimina los accesos en menos de 24 horas.

Dependencias de la fase: base de usuarios activa de fases anteriores, revisión de seguridad específica de OAuth y políticas de permisos mínimos.

Riesgos de la fase:

- Permisos excesivos que erosionan la confianza. Mitigación: alcance mínimo, explicación previa y revocación inmediata.
- Fragilidad ante cambios de los proveedores externos. Mitigación: cada integración es opcional y el producto sigue completo sin ella.
- Soporte multiplicado por combinaciones de proveedores. Mitigación: catálogo cerrado de integraciones soportadas.

## Fase 7 — Escalamiento

Duración [ESTIMACIÓN]: trabajo continuo por trimestres, solo si las Fases 5 y 6 sostienen retención y economía unitaria.

El objetivo de esta fase es crecer en usuarios y en ingreso sin romper costos, latencia ni confianza. El objetivo se cumple cuando el crecimiento es rentable y el sistema soporta la carga con los mismos estándares del MVP.

Entregables de la fase:

- Plan de capacidad con proyecciones de costo por tramo (crecimiento y escala) y estrategias de optimización.
- Endurecimiento operativo: respaldos probados, monitoreo, respuesta a incidentes y revisión de cumplimiento de protección de datos.
- Evaluación de banca abierta bajo el Decreto 0368 de 2026 como camino voluntario futuro, sin captura de credenciales bancarias bajo ninguna circunstancia.
- Programa de referidos y plan de adquisición con costo de adquisición menor a $50.000 COP por usuario [HIPÓTESIS].
- Hoja de ruta pública de mejoras con criterios de priorización.

Los criterios de salida de esta fase son la compuerta de escala sana (se revisa por ciclo):

- Costo de infraestructura dentro del tramo proyectado para el número real de usuarios [ESTIMACIÓN].
- Retención a 30 días sostenida en al menos 40% durante el crecimiento [HIPÓTESIS].
- Conversión a Premium entre 5% y 8% con precios de hasta $20.000 al mes o alrededor de $80.000 al año [HIPÓTESIS].
- Cero brechas de datos personales sin notificar a la autoridad dentro de 15 días hábiles.

Dependencias de la fase: todas las anteriores, más capacidad operativa y asesoría jurídica colombiana para banca abierta y protección de datos.

Riesgos de la fase:

- Crecer con economía unitaria negativa. Mitigación: cada ciclo revisa costo por usuario y frena adquisición si el margen no aguanta.
- Degradación del rendimiento con el volumen. Mitigación: objetivos de latencia y revisiones de capacidad por tramo.
- Presión por monetizar con datos personales. Mitigación: prohibición permanente de venta de datos; el ingreso viene de suscripciones.

## Compuertas GO, PIVOT y STOP

Toda fase termina en una compuerta con tres salidas posibles. Los umbrales siguientes son vinculantes y se aplican como mínimo en la Fase 3; las demás fases usan sus propios criterios de salida más esta lógica de decisión.

GO significa avanzar. Se declara GO cuando se cumplen todos los criterios de salida de la fase y, en validación con usuarios, se alcanzan los umbrales fuertes: activación de al menos 70% con primera obligación dentro de 24 horas, retención de al menos 40% al día 30, promedio de 8 o más obligaciones por usuario, al menos 60% "muy decepcionado" en la prueba de Sean Ellis y NPS superior a 30 [HIPÓTESIS].

PIVOT significa ajustar con alcance acotado. Se declara PIVOT cuando hay tracción parcial: algunas métricas cerca del umbral, un segmento responde mucho mejor que el promedio, o un ajuste concreto explica el desvío. El PIVOT define qué cambia, qué métrica debe mover y en qué plazo se reevalúa. No se abre la fase siguiente hasta revalidar [DECISIÓN].

STOP significa detener el desarrollo en la dirección actual. Se declara STOP cuando la activación es inferior al 50% o más del 60% responde "nada decepcionado" en la prueba de Sean Ellis, o cuando dos ciclos de PIVOT no mueven las métricas [HIPÓTESIS]. STOP no es fracaso administrativo: es la decisión correcta cuando los datos dicen que el problema, la solución o el segmento no sostienen el producto.

Reglas de la compuerta:

- La decisión queda registrada con fecha en formato DD/MM/AAAA, datos observados y motivos.
- Ningún avance de fase sin acta de compuerta firmada por quien lidera producto.
- Los umbrales no se renegocian durante la medición; solo se revisan entre ciclos con justificación escrita.

## Qué queda fuera y por qué

Esta sección fija los límites del roadmap para proteger el foco. Todo lo listado aquí está fuera de su fase indicada por un motivo explícito, no por olvido.

- OCR e inteligencia artificial fuera del MVP, porque primero hay que validar el hábito manual y porque el costo y la privacidad exigen volumen real antes de invertir. Llegan en las Fases 4 y 5.
- Módulos específicos de suscripciones y cuotas fuera del MVP como entidades propias, porque el MVP los cubre como obligaciones recurrentes con categoría y notas. Se separan en la Fase 4.
- Movimiento de dinero e iniciación de pagos fuera de todo el roadmap, porque Bills no mueve dinero ni se presenta como entidad financiera. Los métodos de pago son solo etiquetas genéricas de registro.
- Integraciones bancarias y captura de credenciales fuera de todo el roadmap, porque están prohibidas por diseño. La banca abierta solo se evalúa como camino voluntario y futuro en la Fase 7, sin captura de credenciales.
- Analíticas avanzadas y detección de cambios de precio fuera del MVP, porque requieren historial acumulado. Llegan en la Fase 5 con reglas deterministas primero.
- Notificaciones push, OAuth de Google, MFA, modo oscuro, lectura de correo y sincronización de calendario fuera del MVP, porque amplían la superficie sin validar el núcleo. Llegan en la Fase 6.
- Aplicación nativa fuera del roadmap actual, porque la web móvil cubre el MVP y la validación. Se evaluará solo con escala demostrada.
- Venta de datos personales y publicidad intrusiva fuera del modelo para siempre, porque destruyen la confianza que el producto necesita. El ingreso previsto viene de suscripciones [HIPÓTESIS].
