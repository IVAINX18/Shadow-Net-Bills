# Shadow-Net Bills: visión del producto

Este documento define la visión, el problema, la solución, el público, la propuesta de valor y las métricas de éxito de Shadow-Net Bills. Es la puerta de entrada a la especificación: cualquier lector sin contexto previo debe entender qué es el producto, a quién sirve y por qué existe.

> Convención de etiquetado: cada dato de mercado, precio o afirmación de conversión lleva una etiqueta entre corchetes. `[DATO]` es un hecho verificado con su fuente. `[HIPÓTESIS]` es una suposición por validar. `[ESTIMACIÓN]` es un cálculo con metodología explícita. `[DECISIÓN]` es una elección de diseño del equipo.

## Visión

Queremos que ninguna persona en Colombia pague una multa por olvidar una factura.

Hoy las obligaciones financieras de un hogar están dispersas: el arriendo se paga por transferencia, la luz llega por correo, el plan del celular se consulta en una app, la tarjeta se paga en otra y el internet se vence en una fecha distinta. Shadow-Net Bills centraliza todas esas obligaciones en un solo lugar, con su fecha de vencimiento, su recurrencia y sus recordatorios, para que el usuario siempre sepa qué debe, cuándo vence y qué ya pagó.

La visión de largo plazo [DECISIÓN] es construir el centro de control de las obligaciones financieras personales en Colombia: primero registro manual con recordatorios (MVP), después importación de facturas con OCR (V1), luego análisis inteligente del gasto (V2) y finalmente integraciones con correo, calendario y finanzas abiertas cuando estén operativas en el país (V3 y futuro).

Lo que no cambia con las fases es el principio: Bills organiza y recuerda, no mueve dinero y no pide credenciales bancarias.

## Problema

El problema central es simple y costoso: las personas olvidan pagar facturas porque sus vencimientos están fragmentados en muchos canales.

Las manifestaciones concretas del problema son:

1. Vencimientos dispersos. Cada proveedor comunica por un canal distinto: correo, SMS, app propia o factura física. Ningún canal muestra el conjunto completo.
2. Recurrencias distintas. Un hogar típico combina obligaciones mensuales (arriendo, servicios), trimestrales (algunos seguros), anuales (impuestos, matrícula) y únicas (multas, citas médicas privadas). Llevar esas frecuencias en la cabeza falla.
3. Sin calendario consolidado. Las billeteras existentes permiten pagar facturas individuales pero no muestran todas las obligaciones del mes en una sola vista con su estado (pendiente, pagada, vencida) [DATO]. Fuente: nequi.com.co, consultado 2026-10-07.
4. Costo del olvido. Un día de mora genera intereses y, en servicios públicos y tarjetas, reportes y recargos. Una multa típica por pago tardío de tarjeta ronda los $20.000 COP [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.
5. Estrés financiero evitable. No saber cuánto se debe en total antes de quincena obliga a revisar cuenta por cuenta. La carga mental recae sobre una sola persona del hogar [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

El contexto colombiano agrava el problema: no existe una banca abierta estable que agregue cuentas de forma confiable [DATO]. Fuente: minhacienda.gov.co, consultado 2026-10-07. Los rieles de pago dominantes son PSE (ACH, sin costo para el pagador) y Bre-B (transferencias inmediatas) [DATO]. Fuente: investigación interna Shadow-Net, 2026-10-07. Las billeteras dominan el pago digital: Nequi reporta entre 27 y 29 millones de usuarios [DATO]. Fuente: nequi.com.co, consultado 2026-10-07. Pero pagar no es lo mismo que organizar: ninguna billetera ofrece seguimiento entre proveedores [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

## Solución

Shadow-Net Bills es una aplicación web que permite registrar cada obligación financiera (nombre, monto en COP, categoría, fecha de vencimiento, frecuencia y método de pago genérico), verla en un panel y un calendario mensual, marcarla como pagada y recibir recordatorios por correo electrónico 3, 1 y 0 días antes del vencimiento [DECISIÓN].

El flujo de valor en el MVP es el siguiente:

```
+----------+     +-----------------+     +------------------+
| Usuario  | --> | Registra        | --> | Ve panel +       |
|          |     | obligación      |     | calendario       |
+----------+     +-----------------+     +------------------+
                          |                       |
                          v                       v
                 +-----------------+     +------------------+
                 | Recibe email    |     | Marca como       |
                 | 3, 1 y 0 días   |     | pagada, guarda   |
                 | antes           |     | historial        |
                 +-----------------+     +------------------+
```

Los componentes del MVP son [DECISIÓN]:

- Autenticación con correo y contraseña.
- CRUD de obligaciones sobre la tabla `obligations` con columna `next_due_date`.
- Las 9 categorías fijas: Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno y Otros.
- Panel con tarjetas de resumen, calendario mensual y resumen del mes.
- Historial de pagos, exportación a CSV y borrado de cuenta.
- Recordatorios por correo con offsets `{3,1,0}` días y métodos de pago genéricos (tarjeta, transferencia, efectivo, débito automático, otro).

La solución es deliberadamente manual-primero [DECISIÓN]: el usuario registra sus obligaciones a mano durante 8 semanas de MVP. La importación automática de facturas con OCR llega en V1, el análisis inteligente en V2 y las integraciones con correo y calendario en V3. Esta secuencia permite validar si el dolor es real antes de invertir en tecnología costosa.

## Qué es y qué NO es Shadow-Net Bills

Esta sección evita malentendidos desde el primer día.

Qué es Shadow-Net Bills:

- Un organizador personal de obligaciones: facturas, suscripciones y pagos recurrentes.
- Un sistema de recordatorios por correo antes de cada vencimiento.
- Un calendario mensual y un resumen de lo que se debe y lo que ya se pagó.
- Una herramienta de contexto colombiano: montos en COP, categorías locales y recordatorios alineados con los ciclos de pago del país.

Qué NO es Shadow-Net Bills:

- NO mueve dinero. No inicia pagos, no se conecta a PSE ni a ningún banco para debitar.
- NO almacena credenciales bancarias ni números completos de tarjetas.
- NO hace raspado de portales bancarios (screen scraping). Cuando las finanzas abiertas estén operativas en Colombia (uso realista esperado desde 2027) [DATO]. Fuente: superfinanciera.gov.co, consultado 2026-10-07, se evaluará la integración por el canal oficial.
- NO es una entidad financiera ni ofrece asesoría financiera automática.
- NO incluye IA ni OCR en el MVP. La extracción de facturas es estrategia V1, documentada en `../architecture/ai-strategy.md`.
- NO promete cumplimiento legal. El tratamiento de datos personales sigue la Ley 1581 de 2012 y se detalla en `../legal/regulatory-compliance.md`.

## Público objetivo

El público objetivo son personas en Colombia, entre 22 y 55 años, que gestionan 5 o más obligaciones mensuales y hoy dependen de recordatorios dispersos (correos, notas, memoria) [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Los segmentos prioritarios son:

1. Hogares que pagan arriendo y servicios. Llevan Vivienda y Servicios públicos más Telecomunicaciones. El olvido típico es el agua o el gas cuando el correo se pierde.
2. Trabajadores con tarjetas y suscripciones. Combinan la categoría Financiero (tarjetas, créditos) con Suscripciones (plataformas, gimnasio, celular). El dolor es pagar intereses por un día de retraso.
3. Madres, padres y cuidadores que centralizan los pagos del hogar. Gestionan Educación (pensión, transporte escolar), Seguros y Gobierno (impuestos, comparendos) además de lo anterior. La carga mental es el dolor principal.

Quedan fuera del público inicial [DECISIÓN]: empresas que necesitan facturación electrónica propia, personas que buscan un agregador bancario automático y usuarios que solo quieren pagar una factura suelta sin organizar nada.

## Personas

Tres personas concretas representan al público. Sus cifras son ilustrativas para diseño [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

### Ana, 34 años, empleada en Bogotá que centraliza su hogar

- Contexto: vive con su pareja y un hijo. Trabaja como auxiliar administrativa. Gana alrededor de $2.800.000 COP mensuales [HIPÓTESIS].
- Obligaciones: 11 al mes. Arriendo $1.200.000 (Vivienda), luz $95.000 y agua $70.000 (Servicios públicos), plan celular $65.000 (Telecomunicaciones), dos suscripciones por $18.900 y $27.900 (Suscripciones), tarjeta $450.000 (Financiero), pensión escolar $520.000 (Educación) y 4 más.
- Dolor: en 2025 pagó tres recargos por olvidar el agua y el gas porque las facturas llegaban al correo de su pareja. Perdió cerca de $60.000 COP en recargos en un trimestre [HIPÓTESIS].
- Comportamiento actual: anota vencimientos en un cuaderno y pone alarmas en el celular, pero las alarmas no dicen el monto ni si ya pagó.
- Qué necesita de Bills: registrar todo una vez, ver el calendario del mes y recibir el correo 3 días antes con el monto exacto.

### Carlos, 27 años, desarrollador en Medellín con tarjetas y suscripciones

- Contexto: soltero, trabaja remoto. Gana alrededor de $4.500.000 COP mensuales [HIPÓTESIS].
- Obligaciones: 9 al mes. Internet $90.000 (Telecomunicaciones), luz $80.000 (Servicios públicos), dos tarjetas que suman $900.000 (Financiero), cinco suscripciones por $120.000 en total (Suscripciones) y el seguro de la moto $65.000 (Seguros).
- Dolor: en junio de 2026 pagó $20.000 COP de interés por pagar la tarjeta un día tarde [HIPÓTESIS]. Sabe que Finku cobra $20.000 COP al mes por organizar finanzas [DATO]. Fuente: finku.co, consultado 2026-10-07, pero le parece caro para lo que necesita.
- Comportamiento actual: usa Nequi para pagar facturas sueltas [DATO]. Fuente: nequi.com.co, consultado 2026-10-07, pero Nequi no le muestra el total del mes ni le avisa antes de cada vencimiento.
- Qué necesita de Bills: un panel con el total pendiente del mes, recurrencias automáticas para suscripciones y el historial para saber qué ya pagó.

### María, 45 años, comerciante en Cali que paga de todo un poco

- Contexto: tiene un local comercial y dos hijos adolescentes. Sus ingresos varían entre $3.000.000 y $5.000.000 COP al mes [HIPÓTESIS].
- Obligaciones: 13 al mes. Local $900.000 (Vivienda), energía del local $180.000 (Servicios públicos), plan celular familiar $150.000 (Telecomunicaciones), crédito $600.000 (Financiero), SOAT y póliza $80.000 (Seguros), pensiones $700.000 (Educación), impuesto predial en cuotas (Gobierno) y 6 más.
- Dolor: mezcla gastos del hogar y del negocio en la misma cabeza. En 2026-01 olvidó el predial y pagó sanción [HIPÓTESIS].
- Comportamiento actual: guarda facturas en papel en una carpeta y revisa el correo los domingos. No usa agregadores bancarios porque la sincronización falla en Colombia [DATO]. Fuente: fintonic.com, consultado 2026-10-07.
- Qué necesita de Bills: categorías claras para separar hogar y negocio, notas por obligación (por ejemplo, número de cuota) y exportación a CSV para su contabilidad simple.

## Propuesta de valor

Se evaluaron cuatro alternativas de propuesta de valor con una escala de 1 a 10 por criterio. Los puntajes son juicio del equipo, no medición [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Los cinco criterios fueron: claridad (se entiende en 3 segundos), diferenciación (suena distinto a la competencia), valor percibido (promete un beneficio por el que se pagaría), facilidad de comunicar (cabe en un titular y se recuerda) y ajuste al mercado colombiano (resuena con el dolor local).

| Alternativa | Claridad | Diferenciación | Valor percibido | Facilidad de comunicar | Mercado colombiano | Total |
| --- | --- | --- | --- | --- | --- | --- |
| A. Nunca más olvides pagar una factura (seleccionada) | 10 | 8 | 9 | 10 | 9 | 46 / 50 |
| B. Tus cuentas, bajo control | 7 | 5 | 6 | 7 | 5 | 30 / 50 (6/10) |
| C. Saber cuánto debes, antes de que te lo cobren | 8 | 7 | 8 | 6 | 7 | 36 / 50 (7/10) |
| D. El control total de tus pagos recurrentes | 7 | 6 | 6 | 5 | 6 | 30 / 50 (6/10) |

La propuesta seleccionada es: **Nunca más olvides pagar una factura**.

La justificación por criterio es:

1. Claridad. La frase describe el resultado exacto: cero olvidos. Las alternativas B y D hablan de "control" sin decir contra qué problema, y la C exige una segunda lectura.
2. Diferenciación. Ninguno de los 10 competidores analizados promete memoria garantizada con recordatorios por offsets fijos `{3,1,0}` y calendario consolidado [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07. La B suena a cualquier app financiera y la D suena a banca.
3. Valor percibido. Evitar un recargo de $20.000 COP justifica por sí solo una suscripción que cueste hasta $20.000 COP al mes [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07. La promesa conecta dolor con dinero ahorrado.
4. Facilidad de comunicar. La frase cabe en un titular, se dice en voz alta sin trabarse y se recuerda después de una sola lectura. La C es larga y la D usa la palabra "recurrentes", poco natural en conversación.
5. Mercado colombiano. El olvido por dispersión de canales es el dolor local específico: PSE y billeteras resuelven el pago pero no la memoria [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07. La A nombra ese dolor; la B y la D podrían ser de cualquier país.

## Diferenciador

El diferenciador de Shadow-Net Bills es una combinación, no una función aislada.

La combinación es: seguimiento de obligaciones con recurrencia real más calendario mensual consolidado más contexto colombiano nativo más recordatorios por correo en `{3,1,0}` días más análisis futuro del gasto. Ningún competidor local reúne esos cinco elementos a un precio accesible [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

Por eso Bills NO se describe como una "app para recordar pagos". Esa frase reduce el producto a alarmas y lo pone a competir con la alarma del celular. Lo que Bills hace distinto es:

- Obligaciones como objeto central, con monto, categoría, frecuencia y siguiente vencimiento (`next_due_date`), no como notas sueltas.
- Recurrencia que genera el siguiente vencimiento al marcar como pagada, para suscripciones e impuestos sin reescribir nada.
- Calendario que muestra el mes completo con estados, no una lista plana de deudas.
- Contexto colombiano desde el día uno: COP, las 9 categorías locales, PSE y Bre-B como realidad de pago externa, sin prometer sincronización bancaria que no existe.
- Recordatorios que llegan solos por correo, con el monto y los días restantes, incluso si el usuario no abre la app.

Cada elemento por separado existe en otro producto. La combinación es lo que ningún competidor ofrece hoy [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

## Principios de producto

Seis principios guían cada decisión del MVP [DECISIÓN]:

1. Manual-primero, magia después. Si algo se puede hacer con un formulario claro, no se usa IA. La tecnología compleja debe ganarse su lugar con demanda validada.
2. La memoria es la función. Todo lo que no ayude a recordar o a saber cuánto se debe, sobra en el MVP.
3. Colombia nativo. Precios en COP con formato $20.000, fechas DD/MM/AAAA, categorías y ciclos de pago locales. Nada de supuestos de otros países.
4. Confianza por restricción. Pedimos pocos datos y nunca pedimos lo sensible: sin credenciales bancarias, sin números de tarjeta, sin raspado. La restricción es una característica.
5. Calma financiera. El tono es sobrio y tranquilizador: avisar a tiempo, no alarmar. Sin gamificación ansiosa ni contadores rojos agresivos.
6. Validación antes de escala. Ninguna fase avanza por calendario; cada fase exige su compuerta de métricas. Ver `roadmap.md` para las compuertas GO, PIVOT y STOP.

## Métricas de éxito

Todas las metas siguientes son objetivos por validar, no resultados [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07. Los umbrales vienen de la decisión de producto aprobada el 07/10/2026.

| Métrica | Meta [HIPÓTESIS] | Qué indica |
| --- | --- | --- |
| Activación (primera obligación en 24 horas) | 70% de los registrados | El registro manual no bloquea |
| Retención a 30 días | 40% de los activados | El hábito de revisar el panel existe |
| Densidad de uso | Promedio de 8 o más obligaciones por usuario | El producto cubre el hogar real |
| Prueba Sean Ellis ("muy decepcionado" si cerrara) | 60% o más | Hay ajuste problema-solución |
| NPS | Mayor a 30 | Los usuarios lo recomiendan |
| Recargos evitados (autorreporte) | Al menos 1 recargo evitado al mes por usuario activo | La promesa "nunca más olvides" se cumple |
| Recordatorios efectivos | 90% de correos entregados antes del vencimiento | El scheduler y el envío funcionan |

Las decisiones programadas son: si las métricas alcanzan o superan las metas, GO a la siguiente fase; si quedan por debajo pero hay señales parciales, PIVOT (cambiar segmento, mensaje o canal); si la activación cae por debajo de 50% o más del 60% dice que no le importaría el cierre, STOP [DECISIÓN].

## Relación con otros documentos

Este documento es el resumen ejecutivo del producto. El detalle vive en los documentos vecinos, todos relativos a esta carpeta:

- El análisis de los 10 competidores y la doble lectura del espacio de mercado están en `competitive-analysis.md`.
- Los requisitos numerados del MVP están en `requirements.md` y los 16 casos de uso en `use-cases.md`.
- Las fases y compuertas de validación están en `roadmap.md` y el plan de entrevistas, encuesta y experimentos en `product-validation.md`.
- La arquitectura, el modelo de datos y la API se definen en `../architecture/architecture.md`, `../architecture/database.md` y `../architecture/api.md`.
