# Análisis competitivo de Shadow-Net Bills en el mercado colombiano

Estado: v1.0, 07/10/2026, autor Shadow-Net Technologies. Documento de investigación informativa para decisiones de producto. Ninguna cifra de este documento constituye asesoramiento financiero o jurídico.

Este documento compara a Shadow-Net Bills con las aplicaciones que hoy atienden, total o parcialmente, la organización de las obligaciones financieras personales en Colombia. Su lector entiende, sin contexto previo, qué hacen los competidores, qué dejan sin resolver y por qué Bills puede ocupar un espacio diferenciado.

## 1. Metodología y fuentes

El análisis se elaboró entre septiembre y octubre de 2026 a partir de fuentes públicas: sitios oficiales de cada competidor, fichas de tiendas de aplicaciones y prensa económica colombiana. No se realizaron entrevistas a usuarios de la competencia ni pruebas de laboratorio; las valoraciones comparativas son juicio de producto del equipo, etiquetadas como tales.

Convenciones de etiquetado usadas en este documento [DECISIÓN]:

- `[DATO]` verificado en fuente pública, siempre acompañado de `Fuente: <nombre o URL>, consultado 2026-10-07`.
- `[HIPÓTESIS]` afirmación por validar con usuarios o experimentos.
- `[ESTIMACIÓN]` cifra calculada por el equipo con metodología explícita.
- `[DECISIÓN]` decisión de producto ya aprobada.

Cuando un dato carece de URL pública verificable se cita como `Fuente: investigación interna Shadow-Net, 2026-10-07`. No se inventa ninguna URL. Los precios en pesos se escriben en formato COP (`$20.000`) y las fechas en formato DD/MM/AAAA.

Fuentes principales: kuanto.co; portafolio.co (artículo de febrero de 2026 sobre PliP); finku.co; ficha de NoSeMePasa en App Store Colombia; nequi.com.co; davivienda.com (información de Daviplata); lulobank.com; compararbanco.com y rankingslatam.com (puntuación APS de Nu); mobillsapp.com; fintonic.com. Todas consultadas el 2026-10-07.

Alcance del análisis: se evalúan 10 competidores directos o adyacentes más 5 menciones en etapa temprana o beta. La comparación se centra en la pregunta que define a Bills: quién ayuda al usuario colombiano a no olvidar pagar sus obligaciones recurrentes y a saber cuánto debe antes de que se lo cobren.

## 2. Contexto del mercado colombiano

Colombia paga sus facturas sobre rieles propios. El sistema PSE de ACH Colombia procesa débitos bancarios en línea sin costo para el pagador y es el medio habitual para pagar servicios públicos, telecomunicaciones e impuestos desde la web de cada emisor [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Bre-B, el sistema de transferencias inmediatas interoperables impulsado por el Banco de la República, opera como el riel de pagos persona a persona y persona a comercio en tiempo real [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07. Su adopción crece, pero su función es mover dinero, no recordar vencimientos ni consolidar obligaciones: nadie abre Bre-B para saber qué facturas vencen esta semana [HIPÓTESIS].

Las billeteras digitales dominan la relación cotidiana con el dinero. Nequi declara entre 27 y 29 millones de usuarios [DATO]. Fuente: nequi.com.co, consultado 2026-10-07. Daviplata mantiene una base masiva con foco rural y en subsidios del Gobierno [DATO]. Fuente: davivienda.com, consultado 2026-10-07. Ambas permiten pagar facturas de emisores grandes (Enel, EPM, Claro y similares), pero ninguna consolida obligaciones de todos los proveedores en un solo calendario con recordatorios programables [DATO]. Fuente: nequi.com.co, consultado 2026-10-07.

No existe open banking estable y operativo en Colombia a la fecha de este análisis [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07. El Decreto 0368/2026 ordena las finanzas abiertas para entidades vigiladas, pero los estándares operativos de la Superintendencia Financiera tienen cronograma previsto para el segundo semestre de 2026 y el uso operativo realista se espera desde 2027 [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07. Consecuencia directa para el producto: cualquier competidor que prometa sincronización bancaria automática en Colombia opera hoy sobre agregación inestable o carga manual de extractos, no sobre una API regulada [HIPÓTESIS].

El resultado es un mercado fragmentado: el usuario paga por PSE, mueve plata por Bre-B, recibe subsidios por Daviplata, ahorra en un neobanco y anota sus vencimientos donde puede. Esa fragmentación es el terreno donde compite Bills.

## 3. Matriz comparativa

Criterios de lectura: "Recordatorios" indica si la app avisa antes del vencimiento; "Suscripciones" si gestiona pagos recurrentes como categoría propia; "Sync bancario" si sincroniza cuentas automáticamente; "Encaje colombiano" si entiende emisores, moneda y rieles locales; "Precio" es el costo del plan relevante; "Valoración" es el juicio de producto del equipo [DECISIÓN].

| App | Recordatorios | Suscripciones | Sync bancario | Encaje colombiano | Precio | Valoración |
| --- | --- | --- | --- | --- | --- | --- |
| Kuanto | No, foco en registro de gasto | No como módulo | No, por diseño | Alto, COP nativo y bot de Telegram | Gratis, premium futuro | Registro ágil, pero no previene olvidos |
| PliP | Parcial, atado a facturas DIAN | Parcial, vía facturación electrónica | Agregación de facturas DIAN | Muy alto, solo Colombia | Freemium | Mejor contexto fiscal, débil fuera de factura electrónica |
| Finku | Básicos | Básico en finanzas personales | Manual y empresarial | Alto, operación colombiana | $20.000 al mes o $200.000 al año [DATO]. Fuente: finku.co, consultado 2026-10-07 | Directo rival en precio, pero disperso en focos |
| NoSeMePasa | Sí, planeador de pagos | Sí, planeador | No | Alto, producto local | Gratis [DATO]. Fuente: App Store CO, consultado 2026-10-07 | Competidor funcional más cercano, pero tracción pequeña |
| Nequi | No, solo comprobantes de pago | No | No, es billetera | Muy alto, 27 a 29 millones de usuarios | Gratis [DATO]. Fuente: nequi.com.co, consultado 2026-10-07 | Donde se paga, no donde se planifica |
| Daviplata | No | No | No, es billetera | Alto, foco rural y subsidios | Gratis [DATO]. Fuente: davivienda.com, consultado 2026-10-07 | Cobertura social, nula planificación |
| Lulo Bank | Alertas de cuenta, no de facturas externas | No | No, es neobanco | Medio alto, CDT al 12,5% EA | Gratis [DATO]. Fuente: lulobank.com, consultado 2026-10-07 | Ahorro atractivo, sin centro de obligaciones |
| Nu Colombia | Alertas de tarjeta, no de facturas externas | No | No, es neobanco | Medio, líder en experiencia con APS 93,2 en Q2 2026 | Gratis [DATO]. Fuente: compararbanco.com, consultado 2026-10-07 | La vara de UX a igualar, sin gestión de vencimientos |
| Mobills | Sí, por correo electrónico | Sí, presupuestos | No en Colombia | Bajo, sin PSE ni banca local | Premium cercano a $63.000 al mes [DATO]. Fuente: mobillsapp.com, consultado 2026-10-07 | Buen recordatorio, precio alto y sin contexto local |
| Fintonic | Alertas de movimientos | Parcial | Inestable en Colombia | Bajo, diseño español | Gratis más comisiones [DATO]. Fuente: fintonic.com, consultado 2026-10-07 | Agregador que falla donde Bills debe funcionar |
| Bills (este proyecto) | Sí, correo a 3, 1 y 0 días [DECISIÓN] | Sí, como obligaciones recurrentes [DECISIÓN] | No en MVP, manual primero [DECISIÓN] | Muy alto, categorías y rieles locales [DECISIÓN] | Gratis más Premium [HIPÓTESIS] hasta $20.000 al mes | Combinación inexistente: obligaciones más recurrencia más calendario más contexto |

## 4. Fichas de competidores

### 4.1 Kuanto

Propuesta: registro de gastos conversacional mediante bot de Telegram con OCR de recibos por inteligencia artificial, aplicación web progresiva gratuita con plan premium futuro [DATO]. Fuente: kuanto.co, consultado 2026-10-07.

Fortalezas: fricción mínima para anotar un gasto, moneda COP nativa y multimoneda, decisión explícita de no sincronizar bancos que reduce superficie de riesgo.

Debilidades: mira hacia atrás (qué gasté) y no hacia adelante (qué vence); sin recordatorios de vencimiento ni calendario de obligaciones.

Oportunidad para Bills: captar al usuario que registra en Kuanto pero sigue olvidando pagar, ofreciendo el paso siguiente natural, la planificación de vencimientos con recordatorios a 3, 1 y 0 días [HIPÓTESIS].

### 4.2 PliP

Propuesta: agregación de facturas electrónicas de la DIAN con categorización automática por inteligencia artificial ("Visión IA" a nivel de línea), modelo freemium limitado a Colombia [DATO]. Fuente: portafolio.co, consultado 2026-10-07.

Fortalezas: el mejor encaje fiscal posible, dato estructurado directo de la fuente oficial, categorización automática real.

Debilidades: solo cubre lo que existe como factura electrónica; arriendos informales, cuotas pactadas por chat, servicios sin facturación electrónica y obligaciones sin documento quedan fuera.

Oportunidad para Bills: posicionarse como la capa manual que cubre todo lo que PliP no ve, con importación de facturas como evolución V1 y no como dependencia del día uno [DECISIÓN].

### 4.3 Finku

Propuesta: finanzas personales y de negocio con planes de $20.000 al mes o $200.000 al año, cerca de 1 millón de usuarios declarados y cifrado AES-256 [DATO]. Fuente: finku.co, consultado 2026-10-07.

Fortalezas: marca instalada en Colombia, precio que ancla la disposición a pagar del mercado y cobertura de negocio además de persona natural.

Debilidades: amplitud dispersa (persona más empresa) frente a la profundidad que exige un centro de obligaciones; sus recordatorios son básicos y el calendario no es el corazón del producto.

Oportunidad para Bills: competir por debajo o igualando su ancla de $20.000 con un producto más profundo en un solo trabajo, no olvidar pagar, y validar el techo de precio del Premium contra esta referencia [HIPÓTESIS].

### 4.4 NoSeMePasa

Propuesta: planeador de pagos gratuito enfocado en que no se pase ninguna fecha, producto colombiano de equipo pequeño [DATO]. Fuente: App Store CO, consultado 2026-10-07.

Fortalezas: es el competidor funcionalmente más cercano a Bills; demuestra que el trabajo existe como categoría.

Debilidades: tracción reducida (posición cercana al puesto 83 en Finanzas en App Store Colombia) [DATO]. Fuente: App Store CO, consultado 2026-10-07. Señal ambigua: valida el problema pero no la disposición a usar una app dedicada.

Oportunidad para Bills: ejecutar mejor lo mismo (calendario mensual, historial de pagos, resumen, exportación) y medir activación real; si NoSeMePasa no retiene, la causa probable es ejecución y no ausencia de demanda, hipótesis a validar en entrevistas [HIPÓTESIS].

### 4.5 Nequi

Propuesta: billetera digital con 27 a 29 millones de usuarios que permite pagar facturas de grandes emisores como Enel, EPM y Claro [DATO]. Fuente: nequi.com.co, consultado 2026-10-07.

Fortalezas: distribución inigualable, costumbre de pago instalada, gratuidad total.

Debilidades: no hace seguimiento entre proveedores, no recuerda lo que no se paga dentro de Nequi y no consolida un calendario de obligaciones.

Oportunidad para Bills: convivir en lugar de competir, Bills recuerda y Nequi cobra; el usuario sigue pagando por PSE o Nequi mientras Bills custodia las fechas [DECISIÓN].

### 4.6 Daviplata

Propuesta: billetera con pago de facturas por código de barras, fuerte en zonas rurales y en dispersión de subsidios, gratuita [DATO]. Fuente: davivienda.com, consultado 2026-10-07.

Fortalezas: alcance poblacional donde la banca tradicional no llega, respaldo de Davivienda.

Debilidades: experiencia de planificación inexistente; su usuario objetivo prioriza recibir y pagar puntualmente, no gestionar un portafolio de obligaciones.

Oportunidad para Bills: segmento rural y de subsidios como expansión futura, no como público inicial; el MVP apunta a usuarios urbanos con 5 o más obligaciones mensuales [HIPÓTESIS].

### 4.7 Lulo Bank

Propuesta: neobanco con CDT al 12,5% EA como gancho de ahorro, sin centro de facturas, gratuito [DATO]. Fuente: lulobank.com, consultado 2026-10-07.

Fortalezas: tasa visible y producto bancario real que genera confianza.

Debilidades: resolver el ahorro no resuelve el olvido; ninguna función recuerda vencimientos externos.

Oportunidad para Bills: el usuario que optimiza su CDT en Lulo es el mismo perfil que sufre multas por olvido; el argumento de ahorro por multas evitadas (multa promedio cercana a $20.000 [HIPÓTESIS]) financia mentalmente el Premium.

### 4.8 Nu Colombia

Propuesta: neobanco con la mejor experiencia de usuario del mercado (puntaje APS de 93,2 en Q2 2026), sin centro de pagos de facturas, gratuito [DATO]. Fuente: compararbanco.com, consultado 2026-10-07.

Fortalezas: vara de diseño y simplicidad que todo producto financiero colombiano debe igualar; base de usuarios jóvenes afines al público de Bills.

Debilidades: su recordatorio cubre la tarjeta Nu, no las obligaciones con terceros.

Oportunidad para Bills: tomar a Nu como referencia de UX (menos pantallas, lenguaje simple, estados vacíos amables) sin intentar replicar su licencia bancaria; competir en experiencia, no en balance [DECISIÓN].

### 4.9 Mobills

Propuesta: presupuestos con recordatorios de facturas por correo, plan premium cercano a $63.000 al mes, origen brasileño sin sincronización bancaria colombiana ni PSE [DATO]. Fuente: mobillsapp.com, consultado 2026-10-07.

Fortalezas: recordatorios y presupuestos maduros, años de iteración en Brasil.

Debilidades: precio tres veces superior al ancla local de $20.000 y encaje colombiano nulo; el usuario paga en dólares de facto por un producto que no conoce a EPM ni a Claro Colombia.

Oportunidad para Bills: Colombia primero con precio local; cada peso por debajo de $63.000 es argumento comercial directo [HIPÓTESIS].

### 4.10 Fintonic

Propuesta: agregador bancario español, gratuito con comisiones, cuya sincronización falla o es inestable en Colombia [DATO]. Fuente: fintonic.com, consultado 2026-10-07.

Fortalezas: categorización automática cuando la sincronización funciona; experiencia europea pulida.

Debilidades: dependencia total de una agregación que no opera de forma estable sin open banking local; promesa rota en el mercado objetivo.

Oportunidad para Bills: la estrategia manual primero convierte la debilidad de Fintonic en argumento, donde la sincronización falla, el registro manual con recordatorios sí funciona [DECISIÓN].

## 5. Menciones adicionales

Cinco proyectos merecen vigilancia sin ficha completa por tracción limitada o estado beta a 07/10/2026.

Finthy permite cargar extractos y declara compatibilidad con más de 40 bancos colombianos [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07. Su enfoque en carga de extractos lo acerca a Bills por el lado del historial, pero no ofrece recordatorios de vencimiento como centro del producto [HIPÓTESIS].

Flin es un gestor financiero local en etapa temprana sin información pública suficiente para evaluarlo a fondo [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Mi Bolsillito atiende organización básica de bolsillo con alcance limitado documentado [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Novac aparece como propuesta emergente de finanzas personales sin tracción verificable [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Shushi permanece en beta y su propuesta pública es insuficiente para una ficha completa [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Ninguna de estas menciones combina hoy recordatorios programables, calendario mensual y contexto colombiano [HIPÓTESIS]. Se revisarán de nuevo en la Fase 3 del roadmap.

## 6. Amenazas

Primera amenaza: que Nequi o Daviplata añadan un calendario de vencimientos con recordatorios usando su distribución de decenas de millones de usuarios; Bills no puede competir en alcance, solo en profundidad [HIPÓTESIS].

Segunda amenaza: que PliP extienda su agregación DIAN hacia recordatorios y cubra por defecto a todo facturador electrónico, cerrando la ventana de diferenciación fiscal [HIPÓTESIS].

Tercera amenaza: la llegada del open banking operativo desde 2027 habilita a Fintonic y a neobancos a sincronizar cuentas y lanzar centros de facturas con datos reales, erosionando la ventaja del enfoque manual [HIPÓTESIS].

Cuarta amenaza: NoSeMePasa u otro actor pequeño itera más rápido con menos costo y ocupa el nicho antes de que Bills lance [HIPÓTESIS].

Quinta amenaza: sensibilidad al precio; con anclas gratuitas dominantes (billeteras) y un techo cercano a $20.000, el Premium debe justificarse con valor demostrable en multas evitadas [HIPÓTESIS].

## 7. Oportunidades

Primera oportunidad: ser la capa de planificación que las billeteras no quieren construir; Bills no mueve dinero y por eso puede ser neutral entre PSE, Bre-B, Nequi y efectivo [DECISIÓN].

Segunda oportunidad: precio accesible frente a Mobills y alineado con Finku; el diferencial de hasta $43.000 mensuales contra el premium brasileño es mensaje comercial directo [ESTIMACIÓN]. Metodología: $63.000 menos $20.000 igual a $43.000 de ahorro mensual hipotético.

Tercera oportunidad: contexto colombiano nativo (9 categorías fijas, emisores locales, recordatorios a 3, 1 y 0 días) que ningún competidor extranjero replica [DECISIÓN].

Cuarta oportunidad: entrada manual como virtud de privacidad; no pedir credenciales bancarias genera confianza en un mercado con alta sensibilidad al fraude [HIPÓTESIS].

Quinta oportunidad: evolución V1 con OCR e importación cuando el open banking madure, usando la base manual como datos de entrenamiento propios [DECISIÓN].

## 8. Hipótesis del espacio de mercado

Enunciado de la hipótesis, no hecho: ningún competidor combina hoy seguimiento de obligaciones, recordatorios programables, escaneo de facturas y contexto colombiano a precio accesible [HIPÓTESIS]. Esta sección presenta las dos lecturas obligatorias y el plan para distinguirlas.

Interpretación A, oportunidad real: el espacio está vacío porque los actores existentes miran a otro trabajo (mover dinero, ahorrar, registrar gasto, agregar facturas fiscales) y nadie se apropió del trabajo de recordar. La evidencia a favor es la fragmentación descrita en la sección 2 y la existencia de NoSeMePasa como prueba de que alguien ya nombró la categoría. Si esta lectura es correcta, un MVP manual con recordatorios por correo debería activar al 70% de los registros con su primera obligación en 24 horas [HIPÓTESIS].

Interpretación B, ausencia de demanda: el espacio está vacío porque los usuarios no quieren otra aplicación para sus facturas; prefieren pagar cuando les llega el cobro por WhatsApp, correo o la app de cada emisor, y el olvido ocasional les cuesta menos que gestionar una app. La evidencia a favor es la tracción pequeña de NoSeMePasa y el dominio de billeteras gratuitas que resuelven el pago sin planificar. Si esta lectura es correcta, la activación caerá por debajo del 50% y la respuesta "no me decepcionaría perderla" superará el 60% en la prueba de Sean Ellis [HIPÓTESIS].

Cómo validarla antes de construir de más: primero, 18 entrevistas con el guion del plan de validación para medir intensidad del dolor y soluciones actuales; segundo, experimento concierge con 20 usuarios gestionados manualmente para observar si los recordatorios evitan multas reales; tercero, compuertas GO, PIVOT y STOP del roadmap con umbrales exactos (retención 40% a 30 días, promedio 8 o más obligaciones por usuario, 60% de "muy decepcionado", NPS mayor a 30). Si las tres señales apuntan a la interpretación B, la decisión es STOP o PIVOT, no perseverar [DECISIÓN].

Conclusión operativa: tratar el espacio como hipótesis valiosa pero no probada, construir solo el MVP manual de 8 semanas y dejar que las métricas de validación elijan entre A y B [DECISIÓN].
