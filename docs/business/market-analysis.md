# Análisis de mercado: Shadow-Net Bills en Colombia

Este documento separa de forma estricta lo verificado, lo supuesto y lo estimado para Shadow-Net Bills, una aplicación para organizar y recordar obligaciones financieras personales en Colombia. Todo número de mercado, precio o conversión lleva etiqueta canónica. Nada de lo estimado se presenta como investigación.

## 1. Convenciones de etiquetado

Este documento usa cuatro etiquetas canónicas en texto plano, siempre entre corchetes y en la misma línea que la afirmación que califican.

- [DATO]: hecho verificado con fuente explícita. Formato obligatorio: [DATO] ... Fuente: <nombre o URL>, consultado 2026-10-07. Si no existe URL pública, se escribe Fuente: investigación interna Shadow-Net, 2026-10-07.
- [HIPÓTESIS]: supuesto de producto o mercado que aún no está validado y que debe comprobarse con entrevistas, encuestas o experimentos antes de tomarlo como cierto.
- [ESTIMACIÓN]: cálculo numérico con metodología explícita en la misma sección. Ninguna estimación es un dato. Cada paso del cálculo TAM/SAM/SOM lleva su propia etiqueta [ESTIMACIÓN].
- [DECISIÓN]: decisión de diseño o alcance derivada de los datos, hipótesis o estimaciones anteriores.

Las palabras "hipótesis" o "estimación" pueden aparecer en la prosa, pero la etiqueta entre corchetes es la forma canónica y rastreable con grep.

Alcance temporal de este análisis: datos recolectados entre septiembre y octubre de 2026, consolidados el 07/10/2026.

## 2. Contexto Colombia: pagos, billeteras y banca abierta

Colombia combina alta adopción de billeteras digitales con rieles de pago locales y sin banca abierta operativa a la fecha de este análisis.

Las billeteras dominan el pago cotidiano. [DATO] Nequi reporta entre 27 y 29 millones de usuarios y permite pagar facturas de empresas como Enel, EPM y Claro, pero no ofrece seguimiento consolidado entre proveedores. Fuente: nequi.com.co, consultado 2026-10-07. [DATO] Daviplata ofrece pago de facturas con código de barras y tiene foco rural y de subsidios, con servicio gratuito. Fuente: davivienda.com, consultado 2026-10-07. En conjunto, las billeteras superan los 25 millones de usuarios activos en el país. [DATO] Fuente: investigación interna Shadow-Net, 2026-10-07.

Los rieles de pago relevantes son PSE y Bre-B. [DATO] PSE (ACH Colombia) es el riel de débito bancario en línea, gratuito para el pagador en la mayoría de comercios, y Bre-B es el sistema de transferencias inmediatas interoperables. Fuente: investigación interna Shadow-Net, 2026-10-07. Esto importa para Bills porque el usuario ya tiene cómo pagar; lo que le falta es acordarse de qué pagar, cuándo y por cuánto. Bills no mueve dinero y no necesita conectarse a estos rieles en el MVP.

No existe banca abierta (open banking) operativa y estable en Colombia a la fecha de este análisis. [DATO] El Decreto 0368/2026 de abril de 2026 vuelve obligatorias las finanzas abiertas para entidades vigiladas por la Superintendencia Financiera, con estándares esperados en el segundo semestre de 2026 y uso operativo realista desde 2027. Fuente: minhacienda.gov.co, consultado 2026-10-07. [DATO] Las entidades no vigiladas pueden vincularse de forma voluntaria posterior como Tercero Receptor No Vigilado según la Circular Externa 006/2025 de la Superintendencia Financiera. Fuente: superfinanciera.gov.co, consultado 2026-10-07. [DECISIÓN] Por esta razón el MVP de Bills es manual-primero, sin sincronización bancaria, sin almacenamiento de credenciales y sin screen scraping de bancos bajo ninguna circunstancia.

La adopción digital es alta en zonas urbanas y desigual en zonas rurales. [DATO] La penetración de teléfonos inteligentes supera el 70% en población adulta urbana según reportes sectoriales recurrentes. Fuente: investigación interna Shadow-Net, 2026-10-07. [HIPÓTESIS] El segmento urbano con teléfono inteligente, cuenta bancaria o billetera, y cinco o más obligaciones mensuales es el público inicial más receptivo a Bills.

| Factor de contexto | Estado a 07/10/2026 | Implicación para Bills |
| --- | --- | --- |
| Billeteras digitales | 25M+ usuarios (Nequi 27-29M) | El pago ya está resuelto; el recordatorio no |
| PSE / Bre-B | Rieles operativos y gratuitos o de bajo costo | Bills no necesita mover dinero |
| Banca abierta | Normada pero no operativa (uso realista 2027+) | MVP manual; integración solo en fase futura |
| Adopción smartphone urbano | Alta, superior al 70% adulto urbano | Canal móvil viable para recordatorios por correo |

## 3. Datos investigados

Solo hechos verificados con fuente. Ninguna fila de esta sección es una proyección.

### 3.1 Competidores directos e indirectos verificados

| Aplicación | Origen | Propuesta verificada | Precio verificado |
| --- | --- | --- | --- |
| Kuanto | Colombia | Registro de gastos con bot de Telegram e IA para lectura de recibos, PWA, sin sincronización bancaria por diseño, multimoneda con COP nativo | Gratis (premium planeado) |
| PliP | Colombia | Agregación de facturas electrónicas DIAN con categorización por IA ("Visión IA" a nivel de línea) | Freemium, solo Colombia |
| Finku | Colombia | Finanzas personales y de negocio, ~1M de usuarios declarados, cifrado AES-256 | $20.000 COP/mes o $200.000 COP/año |
| NoSeMePasa | Colombia | Planeador de pagos, gratuita, tamaño pequeño (alrededor del puesto 83 en Finanzas en App Store CO) | Gratis |
| Nequi | Colombia | Billetera con pago de facturas, sin seguimiento entre proveedores | Gratis |
| Daviplata | Colombia | Billetera con pago de facturas por código de barras, foco rural y subsidios | Gratis |
| Lulo Bank | Colombia | Neobanco con CDT al 12,5% EA, sin centro de facturas | Gratis |
| Nu Colombia | Colombia | Neobanco líder en experiencia (puntaje APS 93,2 en Q2 2026), sin centro de pago de facturas | Gratis |
| Mobills | Brasil | Presupuestos y recordatorios de facturas por correo, sin sincronización con bancos colombianos ni PSE | ~$63.000 COP/mes premium |
| Fintonic | España | Agregador bancario cuya sincronización falla o es inestable en Colombia | Gratis más comisiones |

[DATO] Kuanto es un registrador de gastos colombiano con bot de Telegram, lectura de recibos por IA, plan gratuito y PWA, sin sincronización bancaria por diseño. Fuente: kuanto.co, consultado 2026-10-07. [DATO] PliP agrega facturas electrónicas de la DIAN con categorización por IA y modelo freemium solo para Colombia. Fuente: portafolio.co (febrero de 2026), consultado 2026-10-07. [DATO] Finku declara cerca de 1M de usuarios, cifrado AES-256 y precio de $20.000 COP/mes o $200.000 COP/año. Fuente: finku.co, consultado 2026-10-07. [DATO] NoSeMePasa es un planeador de pagos gratuito de tamaño pequeño, ubicado alrededor del puesto 83 en Finanzas en App Store CO. Fuente: App Store CO, consultado 2026-10-07. [DATO] Nequi tiene 27-29M de usuarios y permite pagar facturas sin seguimiento entre proveedores. Fuente: nequi.com.co, consultado 2026-10-07. [DATO] Daviplata permite pagar facturas con código de barras con foco rural y de subsidios. Fuente: davivienda.com, consultado 2026-10-07. [DATO] Lulo Bank es un neobanco con CDT al 12,5% EA y sin centro de facturas. Fuente: lulobank.com, consultado 2026-10-07. [DATO] Nu Colombia lidera en experiencia de usuario con puntaje APS 93,2 en Q2 2026 y no ofrece centro de pago de facturas. Fuente: compararbanco.com, consultado 2026-10-07. [DATO] Mobills ofrece presupuestos y recordatorios por correo con premium de ~$63.000 COP/mes, sin sincronización con bancos colombianos ni PSE. Fuente: mobillsapp.com, consultado 2026-10-07. [DATO] Fintonic es un agregador bancario gratuito cuya sincronización es inestable o falla en Colombia. Fuente: fintonic.com, consultado 2026-10-07.

Menciones adicionales sin ficha completa: Finthy (carga de extractos, declara soporte de más de 40 bancos colombianos), Flin, Mi Bolsillito, Novac y Shushi (beta). Fuente: investigación interna Shadow-Net, 2026-10-07.

### 3.2 Anclas de disposición a pagar verificadas

[DATO] Finku cobra $20.000 COP/mes por su plan pago. Fuente: finku.co, consultado 2026-10-07. [DATO] Mobills cobra el equivalente a ~$63.000 COP/mes por su premium. Fuente: mobillsapp.com, consultado 2026-10-07. Los precios de servicios de suscripción de entretenimiento (música ~$18.900 COP/mes, video ~$50.000 COP/mes) y el valor típico de una multa por pago tardío (~$20.000 COP) se usan solo como anclas de referencia, no como evidencia de disposición a pagar por Bills. [HIPÓTESIS] Fuente: investigación interna Shadow-Net, 2026-10-07.

## 4. Hipótesis de mercado

Toda afirmación de esta sección es un supuesto por validar. Ninguna es un dato.

[Hipótesis H1] [HIPÓTESIS] Las personas con cinco o más obligaciones mensuales olvidan al menos un vencimiento por trimestre y pagan multas o intereses evitables. Validación propuesta: encuesta de 10 preguntas y entrevistas sobre olvidos y multas pagadas en los últimos 90 días.

[Hipótesis H2] [HIPÓTESIS] Los recordatorios por correo electrónico enviados 3, 1 y 0 días antes del vencimiento reducen los pagos tardíos frente a no usar recordatorios. Validación propuesta: experimento concierge con 20 usuarios durante 60 días midiendo tasa de pagos a tiempo.

[Hipótesis H3] [HIPÓTESIS] Existe espacio para Bills: ninguna aplicación colombiana combina seguimiento de obligaciones, recurrencia, calendario mensual, contexto local (categorías colombianas, montos en COP, recordatorios 3/1/0 días) y precio accesible en un solo producto. Esta hipótesis del espacio de mercado admite AMBAS interpretaciones y no se afirma como hecho.

Interpretación A, oportunidad real: el hueco existe porque los competidores están ocupados en otros problemas (billeteras en mover dinero, neobancos en crédito y captación, registradores en gasto pasado) y nadie se especializó en el vencimiento futuro. Si esta interpretación es correcta, un MVP manual enfocado en recordatorios debería activar al 70% de los registros con su primera obligación en 24 horas y retener al 40% a 30 días.

Interpretación B, ausencia de demanda: el hueco existe porque los usuarios no sienten suficiente dolor como para adoptar una aplicación aparte; les basta con el correo del banco, el mensaje de la empresa de servicios o la alarma del celular. Si esta interpretación es correcta, la activación caerá por debajo del 50%, más del 60% dirá que no le molestaría que Bills desapareciera (prueba Sean Ellis invertida) y la disposición a pagar declarada no se convertirá en registros. La ausencia de competidores sería entonces una señal de mercado ausente, no de oportunidad desatendida.

Cómo distinguirlas antes de construir de más: [DECISIÓN] ejecutar en la Fase 0 tres experimentos baratos en este orden: prueba de página de aterrizaje con botón de registro (mide intención), concierge manual con 20 usuarios por WhatsApp y correo (mide uso real sin código), y puerta falsa del botón "Escanear factura" (mide demanda de OCR antes de construirlo). Criterios: GO si activación mayor o igual a 70%, retención a 30 días mayor o igual a 40%, promedio de 8 o más obligaciones por usuario, 60% o más "muy decepcionados" si Bills desapareciera y NPS mayor a 30; PIVOT si los valores quedan en zona intermedia; STOP si activación menor a 50% o más del 60% declara que no le importaría la desaparición. El detalle operativo de estos experimentos vive en el plan de validación de producto.

[Hipótesis H4] [HIPÓTESIS] Entre el 5% y el 8% de los usuarios activos convertirían a un plan Premium de hasta $20.000 COP/mes o ~$80.000 COP/año cuando el OCR (V1) y el análisis (V2) estén disponibles. El precio y la conversión son hipótesis, no hechos, y solo se comprometen después de la validación de disposición a pagar.

[Hipótesis H5] [HIPÓTESIS] El contexto colombiano (categorías locales, montos en COP, recordatorios por correo que funcionan sin instalar nada) diferencia más que una lista larga de funciones genéricas. El diferenciador es la combinación de obligaciones, recurrencia, calendario, contexto y recordatorios, no ser "una aplicación para recordar pagos".

## 5. Estimaciones TAM, SAM y SOM con metodología paso a paso

Método general: embudo descendente con supuestos explícitos en cada paso. Cada paso lleva su etiqueta [ESTIMACIÓN]. Las cifras base de población son redondeos de trabajo, no censos. Ningún resultado de esta sección es un dato investigado.

Supuesto base del ticket: plan Premium de hasta $20.000 COP/mes, equivalente a $240.000 COP/año por usuario pago. Este valor es [HIPÓTESIS] y se usa solo como insumo aritmético del cálculo.

Paso 1, población nacional. [ESTIMACIÓN] Metodología: partir de ~52 millones de habitantes en Colombia como cifra redonda de trabajo. Resultado: 52.000.000 personas. Esta cifra es un redondeo operativo, no un dato censal citado.

Paso 2, población adulta. [ESTIMACIÓN] Metodología: aplicar 70% de adultos sobre la población total (52.000.000 x 0,70). Resultado: ~36.400.000 adultos. El 70% es un supuesto de trabajo declarado, no un dato.

Paso 3, adultos con teléfono inteligente y acceso a banca o billetera. [ESTIMACIÓN] Metodología: aplicar 60% sobre los adultos (36.400.000 x 0,60) como proxy de digitalización urbana y tenencia de cuenta o billetera. Resultado: ~21.840.000 personas digitalizadas. El 60% es un supuesto declarado.

Paso 4, personas con cinco o más obligaciones mensuales. [ESTIMACIÓN] Metodología: aplicar 40% sobre la población digitalizada (21.840.000 x 0,40) como proxy del segmento con suficientes vencimientos para sentir el dolor. Resultado: ~8.736.000 personas con alta carga de obligaciones.

Paso 5, personas dispuestas a probar una solución. [ESTIMACIÓN] Metodología: aplicar 15% sobre el paso anterior (8.736.000 x 0,15) como tasa de prueba inicial optimista para una herramienta gratuita. Resultado: ~1.310.400 usuarios potenciales del producto gratuito o freemium.

### 5.1 TAM (mercado total direccionable)

[ESTIMACIÓN] Metodología TAM: multiplicar el resultado del paso 4 (8.736.000 personas con 5+ obligaciones) por el ticket anual hipotético de $240.000 COP. Cálculo: 8.736.000 x $240.000 = ~$2.096.640 millones COP/año (~2,1 billones COP/año). Este TAM representa el techo teórico si cada persona del segmento pagara el Premium, lo cual no ocurrirá; sirve solo como cota superior.

### 5.2 SAM (mercado disponible servible)

[ESTIMACIÓN] Metodología SAM: restringir el TAM al segmento urbano digitalizado alcanzable con un producto en español, solo web móvil y recordatorios por correo, estimado en 40% del TAM. Cálculo: $2.096.640 millones x 0,40 = ~$838.656 millones COP/año. El 40% es un supuesto de alcance declarado: excluye población rural sin conectividad estable, usuarios que solo operan en efectivo sin correo electrónico y empresas.

### 5.3 SOM (mercado obtenible servible a 3 años)

[ESTIMACIÓN] Metodología SOM: aplicar una captura acumulada de 0,5% del SAM en 3 años para un equipo pequeño sin pauta masiva, apoyado en crecimiento orgánico y referidos. Cálculo: $838.656 millones x 0,005 = ~$4.193 millones COP/año. En usuarios: 0,5% de (8.736.000 x 0,40 = 3.494.400) = ~17.472 usuarios pago, equivalentes a ~$4.193 millones COP/año al ticket hipotético. La tasa de 0,5% es un supuesto prudente declarado.

| Nivel | Universo | Ingreso anual hipotético | Naturaleza |
| --- | --- | --- | --- |
| TAM | ~8.736.000 personas | ~$2.096.640 millones COP/año | Techo teórico |
| SAM | ~3.494.400 personas | ~$838.656 millones COP/año | Alcanzable con producto actual |
| SOM 3 años | ~17.472 usuarios pago | ~$4.193 millones COP/año | Captura prudente 0,5% del SAM |

Advertencia explícita: si la Interpretación B de la sección 4 es la correcta (ausencia de demanda), el SOM real tiende a cero sin importar lo que diga esta tabla. Por eso las compuertas GO/PIVOT/STOP de la Fase 0 deciden antes de invertir en OCR o análisis.

## 6. Decisiones de diseño derivadas

[DECISIÓN] MVP manual-primero: registro manual de obligaciones con nombre, monto en COP, categoría fija, vencimiento, frecuencia y método de pago genérico (tarjeta, transferencia, efectivo, débito automático, otro), sin integraciones bancarias ni iniciación de pagos, porque la banca abierta no es operativa y el valor inicial está en el recordatorio.

[DECISIÓN] Colombia-primero: las 9 categorías fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros), montos en COP con formato $20.000, zona horaria America/Bogota y recordatorios en los offsets 3, 1 y 0 días responden al contexto local verificado en la sección 2.

[DECISIÓN] Recordatorios por correo electrónico como único canal del MVP, porque funcionan sin instalar nada, cuestan ~$0,10 USD por cada 1.000 correos con AWS SES y cubren al segmento digitalizado sin exigir notificaciones push nativas.

[DECISIÓN] Precios Free y Premium tratados como [HIPÓTESIS] hasta completar la validación de disposición a pagar; ningún precio se comunica como definitivo en materiales públicos antes de esa validación.

[DECISIÓN] OCR e importación pospuestos a V1 y análisis inteligente a V2, con confirmación humana obligatoria sobre todo dato extraído automáticamente, porque el costo (~$10 USD por cada 1.000 páginas OCR más fracciones de centavo por factura con modelo ligero) solo se justifica con retención comprobada.

## 7. Riesgos de mercado

R1, el hueco es ausencia de demanda. Probabilidad media, impacto alto. Si los usuarios resuelven el problema con alarmas y correos de los emisores, la activación no llegará al 70% y el SOM colapsa. Mitigación: experimentos de Fase 0 con criterio STOP explícito antes de construir V1.

R2, las billeteras agregan recordatorios. Probabilidad media, impacto alto. Nequi o Daviplata podrían añadir un centro de vencimientos aprovechando su base de 25M+ usuarios. Mitigación: especialización en seguimiento entre proveedores y calendario consolidado, que hoy no ofrecen.

R3, sensibilidad al precio. Probabilidad alta, impacto medio. Con el ancla de $20.000 COP/mes de Finku, un Premium por encima de ese valor enfrentará objeciones. Mitigación: mantener el Premium hipotético en o por debajo de $20.000 COP/mes y validar con la encuesta de disposición a pagar antes de fijarlo.

R4, fatiga de recordatorios. Probabilidad media, impacto medio. Demasiados correos generan ignorado o marcado como spam. Mitigación: offsets fijos 3/1/0 días, resumen mensual único y configuración de recordatorios por obligación desde el MVP.

R5, banca abierta que llega tarde o cambia las reglas. Probabilidad baja a media, impacto medio. Si el cronograma de la Superintendencia se retrasa más allá de 2027, la ventaja manual-first se prolonga; si llega antes, los agregadores ganan fuerza. Mitigación: arquitectura con interfaces desacopladas para integraciones futuras sin reescribir el núcleo.

R6, desconfianza en entregar datos financieros. Probabilidad media, impacto medio. El usuario puede temer registrar montos y vencimientos. Mitigación: posicionamiento explícito (Bills no mueve dinero, no guarda credenciales ni números completos de tarjeta), cifrado en tránsito y en reposo, y política de privacidad conforme a la Ley 1581 de 2012.

## 8. Qué falta por validar y próximos pasos

[Hipótesis pendiente] [HIPÓTESIS] La lista de 18 preguntas de entrevista y 10 preguntas de encuesta del plan de validación confirmará o refutará H1 a H5 con evidencia de usuarios reales antes de comprometer precio o alcance de V1.

[DECISIÓN] No se toma ninguna decisión irreversible de precio, posicionamiento pago o inversión en OCR hasta que las compuertas GO/PIVOT/STOP de la Fase 0 emitan veredicto con datos de los tres experimentos. El siguiente documento de negocio (modelo de negocio) desarrolla los tiers Free y Premium como hipótesis y enlaza a este análisis como su base de mercado.

## 9. Fuentes y trazabilidad

Toda fila [DATO] de este documento cita su fuente con fecha de consulta 07/10/2026. Fuentes oficiales y de proveedor usadas: kuanto.co; portafolio.co; finku.co; App Store CO; nequi.com.co; davivienda.com; lulobank.com; compararbanco.com; mobillsapp.com; fintonic.com; minhacienda.gov.co; superfinanciera.gov.co. Donde no existe URL pública se cita Fuente: investigación interna Shadow-Net, 2026-10-07. Ninguna URL fue inventada para este documento. Las estimaciones de la sección 5 declaran su metodología paso a paso y no constituyen investigación.
