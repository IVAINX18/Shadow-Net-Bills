# Plan de validación de producto de Shadow-Net Bills

Este documento es el plan de validación de Shadow-Net Bills: una aplicación para organizar y recordar las obligaciones financieras personales en Colombia con registro manual, recordatorios por correo y calendario mensual. Todo lo que sigue es plan. Ninguna cifra de este documento es un resultado observado. Las etiquetas entre corchetes indican el estado de cada afirmación: `[HIPÓTESIS]` es algo que se cree pero no se ha comprobado, `[ESTIMACIÓN]` es un cálculo con método explícito, `[DATO]` es un hecho verificado con fuente, y `[DECISIÓN]` es una elección de diseño del equipo. Ver también [product-overview.md](./product-overview.md) y [roadmap.md](./roadmap.md).

## Hipótesis del problema

La hipótesis central del problema es la siguiente [HIPÓTESIS]: las personas en Colombia con cinco o más obligaciones mensuales (arriendo, servicios públicos, telecomunicaciones, suscripciones, créditos, seguros, educación e impuestos) olvidan vencimientos porque sus recordatorios están dispersos entre facturas en papel, correos, mensajes de texto, notas mentales y calendarios que nadie revisa. El olvido genera multas, intereses de mora, reconexiones y estrés.

Hipótesis secundarias que se van a comprobar [HIPÓTESIS]:

1. El usuario no sabe cuánto debe en total este mes hasta que ya le cobraron o le cortaron un servicio.
2. Los recordatorios actuales (alarmas del celular, notas adhesivas, memoria) fallan porque no están ligados al vencimiento real ni se repiten con la recurrencia correcta.
3. El usuario prefiere registrar sus obligaciones a mano antes que entregar las claves de su banco a una aplicación.
4. El dolor es más frecuente en hogares que pagan arriendo más tres o más servicios, y en trabajadores independientes con ingresos variables.
5. Una vista mensual tipo calendario más tres recordatorios por correo (3 días, 1 día y el mismo día) reduce los olvidos frente al método actual del usuario.

Lo que invalidaría la hipótesis [DECISIÓN]: si en las entrevistas menos de la mitad de los participantes recuerda un olvido con costo en los últimos 6 meses, o si la mayoría dice que su método actual le funciona bien, el problema no es lo bastante doloroso y se aplica el criterio STOP definido al final de este documento.

## Usuario objetivo

El usuario objetivo es una persona adulta en Colombia, entre 22 y 55 años, que vive en ciudad, usa teléfono inteligente y correo electrónico a diario, y paga entre 5 y 15 obligaciones al mes [HIPÓTESIS]. No es un experto en finanzas. No quiere aprender contabilidad. Quiere una lista clara de lo que debe, cuándo vence y un aviso a tiempo.

Personas de referencia (coherentes con la visión del producto):

| Persona | Perfil | Obligaciones típicas | Frustración actual |
| --- | --- | --- | --- |
| Ana, 29 años, diseñadora independiente en Bogotá | Ingresos variables, paga arriendo y 6 servicios y suscripciones | Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero | Se entera del vencimiento cuando llega el mensaje de mora |
| Carlos, 41 años, empleado con dos hijos en Medellín | Ingresos fijos, paga colegio, créditos y seguros | Vivienda, Educación, Financiero, Seguros, Gobierno | Lleva las fechas en la cabeza y en un cuaderno; su pareja no ve lo mismo |
| María, 34 años, enfermera por turnos en Cali | Horarios rotativos, poco tiempo para trámites | Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Otros | Paga tarde porque el turno le cruza el vencimiento y nadie le avisa |

Criterios de selección para entrevistas y experimentos [DECISIÓN]: residir en Colombia, pagar al menos 4 obligaciones al mes, usar correo electrónico a diario, no trabajar en el equipo del proyecto. Se excluye a quien ya usa una aplicación dedicada de recordatorio de pagos como herramienta principal, porque interesa el dolor no resuelto.

## Alternativas actuales

El usuario resuelve hoy el problema con una mezcla de métodos. Ninguno combina lista de obligaciones con recurrencia, calendario mensual y recordatorios ligados al vencimiento en contexto colombiano [HIPÓTESIS]:

| Alternativa | Cómo se usa | Por qué falla |
| --- | --- | --- |
| Memoria y notas mentales | Recordar las fechas de cabeza | Falla con más de 5 vencimientos o con turnos rotativos |
| Alarmas del celular y calendario genérico | Crear un evento por factura | Hay que crearlo a mano cada mes; no entiende recurrencia trimestral o anual |
| Hojas de cálculo y cuadernos | Apuntar montos y fechas | Nadie lo abre a tiempo; no avisa solo |
| Correos y mensajes de las empresas | Esperar el aviso del proveedor | Llegan tarde, se pierden entre promociones o nunca llegan |
| Billeteras (Nequi, Daviplata) | Pagar cuando se acuerda | Muestran solo sus propios servicios; no hay una vista de todas las obligaciones [DATO] Fuente: nequi.com.co, consultado 2026-10-07 |
| Aplicaciones de gastos (Finku, Mobills) | Registrar gastos generales | Están centradas en presupuesto, no en vencimientos con recordatorio previo [DATO] Fuente: finku.co, consultado 2026-10-07 |
| No hacer nada hasta la mora | Pagar cuando cobran el recargo | Funciona hasta que la multa o el corte duele |

La propuesta de valor seleccionada para comprobar en las entrevistas es "Nunca más olvides pagar una factura" [DECISIÓN]. Si el participante describe su método actual como suficiente y sin fricción, se registra como evidencia contra la hipótesis.

## Intensidad del dolor

La intensidad del dolor se mide en dos ejes: frecuencia (cada cuánto se olvida algo) y severidad (cuánto cuesta el olvido en dinero, tiempo y estrés). Las cifras de esta sección son supuestos de trabajo para diseñar la muestra, no resultados [ESTIMACIÓN].

Estimaciones de partida con método explícito [ESTIMACIÓN]: se supone un usuario con 8 obligaciones mensuales promedio (promedio objetivo del MVP). Con 8 vencimientos al mes, basta una tasa de fallo del 12,5 por ciento mensual para producir en promedio 1 olvido al mes, es decir unos 12 olvidos al año. Con un recargo promedio supuesto de $20.000 COP por olvido (ancla: multas y recargos típicos reportados por usuarios), el costo anual estimado es 12 por $20.000 = $240.000 COP al año. Método: obligaciones por tasa de fallo por recargo promedio. Si la tasa real resulta menor, el plan sigue siendo válido porque la encuesta mide la tasa real.

Severidad no monetaria [ESTIMACIÓN]: se estima que un corte o una reconexión consume entre 1 y 3 horas de trámites (llamadas, filas, pagos presenciales). El estrés se mide en la encuesta con una escala de 1 a 5. Un puntaje promedio igual o mayor a 3,5 en estrés ligado a vencimientos se considera dolor alto y apoya la decisión GO, siempre junto con los umbrales duros de la última sección.

Preguntas que convierten el dolor en número (se aplican en entrevista y encuesta): cuántos olvidos en los últimos 3 meses, cuánto pagó en multas o intereses en ese periodo, cuántas horas perdió en trámites por mora, y qué tan estresante fue en escala de 1 a 5. Sin estos cuatro números no se puede declarar validado el dolor.

## Disposición a pagar

La disposición a pagar es por ahora una hipótesis, no un precio comprometido [HIPÓTESIS]. El modelo previsto es un nivel gratuito (hasta 15 obligaciones, recordatorios, calendario, exportación) y un nivel Premium con hipótesis de precio de hasta $20.000 COP al mes o unos $80.000 COP al año [HIPÓTESIS].

Anclas de precio usadas en las preguntas (todas [HIPÓTESIS] hasta validar):

| Ancla | Valor | Uso en la validación |
| --- | --- | --- |
| Finku | $20.000 COP al mes | Techo de referencia de una aplicación financiera colombiana |
| Multa o recargo típico | $20.000 COP por olvido | Comparación: un mes de Premium cuesta lo mismo que un olvido evitado |
| Spotify | $18.900 COP al mes | Referencia de suscripción de consumo masivo |
| Netflix | $50.000 COP al mes aprox. | Techo blando: por encima de esto la resistencia crece |
| Ahorro anual estimado | $240.000 COP al año | Marco de valor: el plan anual de $80.000 COP se presenta como un tercio del costo estimado de olvidar |

La regla de interpretación es estricta [DECISIÓN]: la disposición declarada en encuesta no basta. Solo cuenta como evidencia la disposición con compromiso (dejar el correo para la beta, aceptar una prueba pagada, o pagar por adelantado en el experimento concierge). La conversión de gratuito a pago del 5 al 8 por ciento es [HIPÓTESIS] y se valida después del MVP, no en esta fase.

## Guion de entrevistas

Objetivo del guion: comprobar el problema antes de hablar de la solución. Reglas para quien entrevista [DECISIÓN]: 30 minutos por sesión, 12 a 15 entrevistas en la primera ronda (fechas de campo propuestas 13/10/2026 a 24/10/2026), no vender el producto, no sugerir respuestas, pedir el último ejemplo concreto en cada pregunta de dolor, y cerrar siempre con un pedido de compromiso. Se redactan 18 preguntas en 5 secciones.

### Sección A: contexto (3 preguntas)

- P1. Cuéntame cómo es un mes normal de pagos en tu casa: qué pagas, quién lo paga y en qué fechas se concentra.
- P2. Qué herramientas usas hoy para no olvidar nada (celular, cuaderno, hoja de cálculo, correos, alarmas) y cuál es la que más usas.
- P3. Cuántas obligaciones pagas al mes entre arriendo, servicios, internet, celular, suscripciones, créditos, seguros, colegio e impuestos. Dame el número aproximado.

### Sección B: descubrimiento del dolor (5 preguntas)

- P4. Cuéntame la última vez que olvidaste pagar algo: qué era, cuándo vencía y cuándo te diste cuenta.
- P5. Qué te costó ese olvido en plata (multa, interés, reconexión) y en tiempo (trámites, llamadas, filas).
- P6. En los últimos 3 meses, cuántas veces pagaste algo tarde aunque sea por un día.
- P7. Del 1 al 5, qué tan estresante es para ti la semana de vencimientos, y qué parte estresa más.
- P8. Qué pasó la vez que más te dolió un olvido: cuánto fue lo máximo que pagaste de más en el último año.

### Sección C: soluciones actuales (4 preguntas)

- P9. Muéstrame o descríbeme paso a paso cómo registras hoy un vencimiento nuevo desde que te llega la factura.
- P10. Qué parte de tu método actual funciona bien y qué parte te gustaría cambiar.
- P11. Has probado alguna aplicación para organizar pagos o gastos. Cuál, cuánto la usaste y por qué la dejaste.
- P12. Cuando la empresa te avisa por correo o mensaje, ese aviso te llega a tiempo o te llega tarde. Dame un ejemplo.

### Sección D: prueba de solución (3 preguntas)

- P13. Si tuvieras una lista con todo lo que debes este mes, con fecha y monto, y te llegara un correo 3 días antes, 1 día antes y el mismo día, qué cambiaría en tu mes.
- P14. Qué no te gusta de esa idea o qué te hace dudar (registro manual, privacidad, otro aviso más, precio).
- P15. Qué tendría que hacer esa lista para que la uses todos los meses y no solo la primera semana.

### Sección E: compromiso (3 preguntas)

- P16. Me dejas tu correo para avisarte cuando exista una versión de prueba y probarías registrar tus obligaciones reales durante 2 semanas.
- P17. Si la prueba te evita una sola multa de $20.000 COP en el mes, pagarías hasta $20.000 COP al mes por seguir usándola. Por qué sí o por qué no.
- P18. A quién más conoces con este mismo problema que aceptaría una entrevista de 30 minutos esta semana. Me compartes su contacto.

Criterio de lectura [DECISIÓN]: el problema se considera validado en entrevistas si al menos 8 de 12 participantes relatan un olvido con costo en los últimos 6 meses y al menos 8 aceptan el compromiso de P16. Por debajo de eso, se pivota el segmento o el encuadre antes de construir.

## Encuesta de validación

La encuesta es el complemento cuantitativo de las entrevistas. Meta del plan: 150 respuestas en Colombia entre el 13/10/2026 y el 31/10/2026 [DECISIÓN]. Se redactan 10 preguntas con opciones cerradas. Cada respuesta alimenta un umbral de decisión.

- E1. Cuántas obligaciones pagas al mes (arriendo, servicios, celular, suscripciones, créditos, seguros, educación, impuestos). Opciones: a) 1 a 3; b) 4 a 6; c) 7 a 10; d) 11 o más. Mide: número de obligaciones. Apunta al promedio de 8 o más del criterio GO.
- E2. En los últimos 3 meses, cuántas veces pagaste algo después del vencimiento. Opciones: a) ninguna; b) 1 vez; c) 2 a 3 veces; d) 4 o más. Mide: olvidos.
- E3. En los últimos 3 meses, cuánto pagaste de más por multas, intereses o reconexiones. Opciones: a) $0 COP; b) menos de $20.000 COP; c) $20.000 a $60.000 COP; d) más de $60.000 COP. Mide: multas pagadas.
- E4. Cómo organizas hoy tus vencimientos (elige el principal). Opciones: a) memoria; b) alarmas o calendario del celular; c) cuaderno u hoja de cálculo; d) aplicación o avisos de la empresa. Mide: método actual.
- E5. Qué tan bien te funciona tu método actual. Opciones: a) muy bien, nunca fallo; b) bien, fallo rara vez; c) regular, fallo a veces; d) mal, fallo seguido. Mide: satisfacción. Si más del 60 por ciento responde a o b, es señal contra la hipótesis.
- E6. Qué tan estresante es la semana de vencimientos para ti (1 a 5). Opciones: a) 1 a 2; b) 3; c) 4; d) 5. Mide: estrés. Promedio igual o mayor a 3,5 apoya el dolor.
- E7. Qué tan útil te sería una lista mensual con fecha y monto más un correo 3 días, 1 día y el mismo día del vencimiento. Opciones: a) nada útil; b) poco útil; c) útil; d) muy útil. Mide: utilidad percibida.
- E8. Pagarías hasta $20.000 COP al mes por una aplicación que te evite al menos una multa al mes. Opciones: a) sí, seguro; b) quizá, depende de la prueba; c) no, prefiero gratis con límites; d) no pagaría nada. Mide: disposición a pagar [HIPÓTESIS].
- E9. Elige las 3 funciones más importantes para ti. Opciones (elige 3): a) registrar obligaciones con recurrencia; b) calendario mensual; c) recordatorios por correo 3, 1 y 0 días; d) resumen mensual de lo que debo; e) exportar a CSV; f) escanear facturas con el celular. Mide: funciones top 3. La opción f es trampa controlada para el experimento fake door.
- E10. Dejas tu correo para entrar a la prueba cerrada de 2 semanas con 20 usuarios. Opciones: a) sí, dejo mi correo; b) sí, pero avísame después; c) no. Mide: compromiso para la beta. Tasa de correos dejados es la métrica de conversión de la encuesta.

## Experimento 1: prueba de página de aterrizaje

Descripción: una página sencilla de una sola pantalla con el titular "Nunca más olvides pagar una factura", tres beneficios (lista mensual con montos, calendario, correos 3 días, 1 día y el mismo día), un ejemplo de calendario ficticio y un único botón "Quiero probarla" que pide el correo. Tráfico supuesto: 500 visitas desde grupos y comunidades colombianas entre el 13/10/2026 y el 27/10/2026 [ESTIMACIÓN] (método: 5 publicaciones por 100 visitas esperadas cada una; si el tráfico real es menor, la métrica se lee en porcentaje, no en absolutos).

Métrica de éxito [DECISIÓN]: conversión de visita a correo igual o mayor al 15 por ciento (75 correos con 500 visitas). Por debajo del 8 por ciento, el mensaje o el segmento falla y se pivota el titular antes de construir. Entre 8 y 15 por ciento, se repite con un titular distinto. Métricas secundarias: clic en el botón (interés) y respuestas a una pregunta abierta de qué factura más temen olvidar (lenguaje del cliente para la página final).

Lo que este experimento no prueba: no prueba retención ni pago. Solo prueba que el mensaje atrae. El pago se prueba en el experimento 2.

## Experimento 2: servicio manual concertado con 20 usuarios

Descripción: un concierge MVP manual con 20 usuarios durante 2 semanas (propuesta 03/11/2026 a 14/11/2026) [DECISIÓN]. No hay aplicación. El equipo registra a mano las obligaciones que cada usuario envía por correo o formulario, arma una tabla mensual por usuario y envía los recordatorios a mano 3 días, 1 día y el mismo día del vencimiento. Cada envío se registra en una hoja de control con fecha y obligación para simular el futuro registro de notificaciones.

Métrica de éxito [DECISIÓN]: activación del 70 por ciento o más (al menos 14 de 20 registran su primera obligación en las primeras 24 horas) y al menos el 60 por ciento declara en la pregunta de Sean Ellis que se sentiría "muy decepcionado" si el servicio dejara de existir. Al cierre se pide el compromiso de pago: preinscripción al plan anual supuesto de $80.000 COP. Si al menos 6 de 20 aceptan la preinscripción, la disposición a pagar deja de ser solo declarada.

Costo y esfuerzo estimados [ESTIMACIÓN]: 20 usuarios por 8 obligaciones promedio son 160 vencimientos simulados; a 5 minutos de gestión por vencimiento son unas 13 horas en 2 semanas, asumibles por una persona. Si el esfuerzo supera las 20 horas, el proceso manual no escala ni para validar y hay que simplificar el protocolo.

## Experimento 3: puerta falsa "Escanear factura"

Descripción: dentro de la página de aterrizaje y del formulario del concierge se muestra un botón "Escanear factura con el celular" que no funciona de verdad. Al pulsarlo aparece un mensaje honesto: "Esta función aún no existe. Déjanos tu correo y te avisamos cuando llegue". El objetivo es medir demanda de OCR antes de construirlo, porque el OCR está fuera del MVP por decisión [DECISIÓN].

Métrica de éxito [DECISIÓN]: si más del 30 por ciento de los visitantes o usuarios pulsa el botón, hay demanda real de escaneo y se prioriza en V1 con el pipeline escalonado (primero lectura gratuita de XML DIAN, luego OCR en la nube, luego modelo de lenguaje solo en baja confianza). Si menos del 10 por ciento lo pulsa, el escaneo no es prioridad y V1 se centra en importación manual rápida. El experimento fake door nunca cobra ni finge un resultado: el mensaje aclara que la función no existe.

## Criterios de decisión GO, PIVOT y STOP

Los umbrales siguientes son los umbrales exactos aprobados para el proyecto y se aplican sobre el MVP real con usuarios, no sobre esta fase de validación previa [DECISIÓN]. Se copian aquí literalmente para que la validación previa use el mismo listón.

| Veredicto | Condición | Qué se hace |
| --- | --- | --- |
| GO | Activación del 70 por ciento (70%) o más (primera obligación registrada en 24 horas o menos) y retención del 40 por ciento (40%) a 30 días y promedio de 8 o más obligaciones por usuario y 60 por ciento (60%) o más "muy decepcionado" en Sean Ellis y NPS mayor a 30 | Avanzar a V1 (OCR e importación) |
| PIVOT | Resultados por debajo de GO pero por encima de STOP (por ejemplo activación entre 50 y 69 por ciento, o retención entre 25 y 39 por ciento, o Sean Ellis entre 40 y 59 por ciento) | Cambiar segmento, mensaje o encuadre y repetir la medición antes de construir más |
| STOP | Activación menor al 50 por ciento o más del 60 por ciento responde "no decepcionado" en Sean Ellis | Detener el proyecto en su forma actual; no construir V1 |

Reglas de lectura [DECISIÓN]: las cinco condiciones de GO se exigen en conjunto para declarar GO. Basta una condición de STOP para declarar STOP aunque las demás se cumplan. La validación previa (entrevistas, encuesta, landing, concierge, fake door) usa los mismos porcentajes como referencia: 70 por ciento de activación (70%), 40 por ciento de retención a 30 días (40%), 8 o más obligaciones, 60 por ciento en Sean Ellis (60%) y NPS mayor a 30. Todo veredicto se registra con fecha en formato DD/MM/AAAA y con el tamaño de muestra al lado, porque un 70 por ciento con 20 usuarios no significa lo mismo que con 500.

## Plan de ejecución y cronograma

El plan se ejecuta en 5 semanas como estimación de trabajo, no como compromiso de calendario [ESTIMACIÓN]. Semana 1 (13/10/2026 a 17/10/2026): entrevistas P1 a P18 y publicación de la página de aterrizaje. Semana 2 (20/10/2026 a 24/10/2026): cierre de entrevistas y encuesta E1 a E10 hasta 150 respuestas. Semana 3 (27/10/2026 a 31/10/2026): lectura de resultados contra los umbrales de referencia y decisión de lanzar o no el concierge. Semanas 4 y 5 (03/11/2026 a 14/11/2026): concierge con 20 usuarios más fake door. Semana 6 (17/11/2026): veredicto GO, PIVOT o STOP por escrito.

Responsables y salidas [DECISIÓN]: una persona conduce entrevistas, otra analiza la encuesta, y ambas firman el veredicto. Salidas exigidas: transcripciones anonimizadas, hoja de cálculo de la encuesta, lista de correos de la página con su tasa de conversión, hoja de control del concierge con activación y Sean Ellis, y conteo de clics del fake door. Sin estas cinco salidas no hay veredicto válido. Este plan no presenta ningún resultado: la primera medición real se escribe solo después de ejecutar lo aquí descrito.
