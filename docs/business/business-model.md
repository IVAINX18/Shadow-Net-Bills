# Modelo de negocio de Shadow-Net Bills

Fecha: 07/10/2026. Estado: hipótesis de trabajo, no hechos validados. Todas las cifras de mercado, precio y conversión llevan etiqueta ([HIPÓTESIS], [ESTIMACIÓN] o [DATO]) según la convención del plan. Nada de lo aquí descrito compromete precios finales; todo precio se valida antes de publicarse.

## 1. Principios del modelo

El modelo de negocio de Shadow-Net Bills se sostiene sobre tres principios que no se negocian [DECISIÓN]:

1. Suscripción simple y transparente. El usuario paga por el producto, no con sus datos. Un solo plan de pago, sin niveles confusos ni cargos ocultos.
2. NUNCA venta de datos. Los datos financieros del usuario no se venden, no se alquilan y no se comparten con terceros con fines comerciales. La venta de datos queda fuera del modelo hoy y en cualquier versión futura [DECISIÓN].
3. Precios accesibles para Colombia. El precio debe caber en el presupuesto de un hogar colombiano promedio. Si el precio no es pagable por nuestro usuario objetivo, el modelo está mal diseñado, no el usuario.

Estos principios guían cada decisión de precio, funcionalidad y alianza descrita abajo.

## 2. Tier Gratuito (Free)

El tier gratuito existe para que cualquier persona organice sus obligaciones sin pagar [DECISIÓN]. Límites propuestos [HIPÓTESIS]:

| Característica | Tier Gratuito |
| --- | --- |
| Obligaciones activas | Hasta 15 obligaciones en la tabla `obligations` |
| Recordatorios por correo | Sí, offsets `{3,1,0}` días antes del vencimiento |
| Calendario mensual | Sí |
| Resumen mensual | Sí, básico |
| Exportación CSV | Sí |
| Categorías | Las 9 fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros) |
| OCR e importación de facturas | No (reservado a V1 de pago) |
| Análisis inteligente | No (reservado a V2 de pago) |

El límite de 15 obligaciones es una hipótesis de diseño [HIPÓTESIS]: cubre al usuario ocasional y deja espacio para que el usuario intensivo (8 o más obligaciones promedio según las métricas de validación) encuentre valor en Premium. El número exacto se ajusta con datos de uso reales. Fuente: investigación interna Shadow-Net, 2026-10-07.

## 3. Tier Premium

El tier Premium agrupa las funcionalidades que cuestan dinero operar o que requieren trabajo avanzado [HIPÓTESIS]:

| Característica | Tier Premium |
| --- | --- |
| Obligaciones activas | Ilimitadas |
| Recordatorios por correo | Sí, con prioridad de envío y reintentos |
| Calendario mensual | Sí, con vistas extendidas |
| OCR e importación de facturas (V1) | Sí, con revisión humana obligatoria |
| Análisis inteligente (V2) | Sí, resúmenes y detección de cambios de precio |
| Soporte | Prioritario |

Precio propuesto: hasta $20.000 COP/mes o alrededor de $80.000 COP/año en plan anual [HIPÓTESIS]. Este tope no es un precio publicado; es el techo que se someterá a validación con usuarios antes de comprometerse. Fuente: investigación interna Shadow-Net, 2026-10-07.

## 4. Disposición a pagar y anclas de precio

La disposición a pagar aún no está medida; las siguientes cifras son anclas de contexto, no pruebas de que el usuario pagará [HIPÓTESIS]:

| Ancla | Cifra | Lectura para Bills |
| --- | --- | --- |
| Finku, app colombiana de finanzas personales | $20.000 COP/mes o $200.000 COP/año [DATO]. Fuente: finku.co, consultado 2026-10-07 | Techo Creíble para una app financiera local; Bills no debería superarlo sin una razón fuerte |
| Spotify, plan individual en Colombia | $18.900 COP/mes [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07 | Referencia de gasto digital mensual que el usuario ya acepta |
| Netflix, plan estándar aproximado | ~$50.000 COP/mes [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07 | Techo de entretenimiento; Bills debe costar una fracción de esto |
| Multa o recargo típico por pagar tarde una factura | ~$20.000 COP por evento [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07 | Un solo olvido evitado al mes pagaría la suscripción; es el argumento central de valor |

La tesis es que evitar una multa de ~$20.000 COP al mes justifica una suscripción de hasta $20.000 COP/mes [HIPÓTESIS]. Esta tesis debe comprobarse con entrevistas y con el experimento de disposición a pagar descrito en ../product/product-validation.md antes de fijar precios.

## 5. Estructura de costos

Costos operativos estimados del MVP con la arquitectura aprobada (Supabase Pro + API en Render o Fly + Cloudflare Pages + AWS SES) [ESTIMACIÓN, metodología: suma de precios públicos de cada proveedor para 100 a 500 usuarios]:

| Componente | Costo mensual estimado |
| --- | --- |
| Supabase Pro (PostgreSQL, Auth, Storage) | $25/mes [DATO]. Fuente: supabase.com/pricing, consultado 2026-10-07 |
| API (Render ~$25/mes o Fly ~$4-7/mes) | $5 a $25/mes [DATO]. Fuente: render.com/pricing y fly.io/pricing, consultado 2026-10-07 |
| Cloudflare Pages y CDN | $0/mes, plan gratuito [DATO]. Fuente: cloudflare.com, consultado 2026-10-07 |
| AWS SES (~$0,10 por cada 1.000 correos) | $1 a $5/mes según volumen [DATO]. Fuente: aws.amazon.com, consultado 2026-10-07 |
| Total MVP | $30-55/mes [ESTIMACIÓN] |

Costo de IA a escala (solo V1 en adelante, fuera del MVP): alrededor de $0,10/usuario/mes con el pipeline escalonado (parseo XML DIAN gratuito, luego OCR a ~$10 por cada 1.000 páginas, luego LLM solo en baja confianza) [ESTIMACIÓN, metodología: 10 facturas por usuario al mes por costo objetivo menor a $0,01 por factura]. Fuente de precios base: páginas de precios de Azure Document Intelligence, AWS Textract y Google Document AI, consultado 2026-10-07.

Conclusión: el MVP cuesta entre $30-55/mes de infraestructura fija, y la IA añade un costo variable cercano a $0,10/usuario/mes cuando se active en V1 [ESTIMACIÓN].

## 6. Unit economics

Todos los indicadores de esta sección son hipótesis de planificación, no hechos [HIPÓTESIS]. Metodología explícita por indicador:

| Indicador | Hipótesis | Metodología |
| --- | --- | --- |
| Conversión Free a Premium | 5-8% de usuarios activos mensuales [HIPÓTESIS] | Rango típico de freemium en apps de consumo citado como referencia de planificación; se mide con cohorte de los primeros 500 usuarios del MVP. Fuente: investigación interna Shadow-Net, 2026-10-07 |
| CAC (costo de adquisición por cliente de pago) | Menor a $50.000 COP [HIPÓTESIS] | Presupuesto de adquisición de los experimentos de validación dividido por clientes convertidos; canales previstos: voz a voz, comunidades y contenido, sin pauta masiva inicial |
| LTV (valor de vida del cliente) | Mayor a $500.000 COP [HIPÓTESIS] | Cálculo: precio mensual hipotético de $20.000 COP por retención supuesta de 25 meses o más; la retención real se mide en la fase de validación con usuarios reales |
| Margen bruto por usuario Premium | Positivo desde el primer mes [HIPÓTESIS] | Ingreso de hasta $20.000 COP/mes menos costo variable (~$0,10/usuario/mes de IA más ~$0,05 de correo estimado); la infraestructura fija de $30-55/mes se diluye con escala |

Regla de control [DECISIÓN]: si la conversión cae por debajo de 5% o el CAC supera $50.000 COP de forma sostenida, el modelo se revisa (precio, paquete Premium o estrategia de adquisición) antes de escalar gasto.

## 7. Qué justificaría pagar Premium

Un usuario pagaría Premium solo si el plan gratuito se le queda corto de forma evidente [HIPÓTESIS]. La lista razonada de justificadores:

1. Más de 15 obligaciones activas. Hogares con vivienda, servicios, telecom, suscripciones y créditos superan rápido el límite gratuito.
2. Ahorro comprobable. Un recordatorio que evita una multa de ~$20.000 COP paga el mes [HIPÓTESIS].
3. OCR que ahorra tiempo real. Fotografiar una factura y tener los datos extraídos con revisión humana vale más que digitar a mano, desde V1 [HIPÓTESIS].
4. Análisis que anticipa gastos. Detectar un aumento de precio antes de que llegue el cobro es información por la que vale pagar, desde V2 [HIPÓTESIS].
5. Tranquilidad y control. Calendario completo, historial exportable y soporte prioritario para quien gestiona las finanzas del hogar.

Ninguno de estos justificadores se asume probado; cada uno se comprueba en entrevistas y en el plan de ../product/product-validation.md.

## 8. Plan anual y precio

Se propone un plan anual equivalente a 10 meses de pago, es decir, 2 meses gratis frente al pago mes a mes [HIPÓTESIS]. Con el techo hipotético de $20.000 COP/mes, el anual rondaría $200.000 COP, con una meta de trabajo cercana a $80.000 COP/año si la validación muestra sensibilidad alta al precio [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

El descuento anual cumple dos funciones: premia el compromiso y mejora el flujo de caja para cubrir la infraestructura fija de $30-55/mes [DECISIÓN]. El porcentaje final de descuento se fija después de medir elasticidad de precio con usuarios reales.

## 9. Lo que NO haremos

Límites explícitos del modelo, sin excepciones [DECISIÓN]:

1. Venta de datos. Nunca venderemos ni alquilaremos los datos financieros de los usuarios. La frase "venta de datos" no aparece en ningún contrato, alianza o integración futura.
2. Publicidad intrusiva. No habrá anuncios de terceros dentro del producto ni mensajes patrocinados disfrazados de recordatorios.
3. Préstamos ni intermediación financiera. Bills no otorga créditos, no cobra comisiones por colocar productos financieros y no se presenta como entidad financiera.
4. Cargos ocultos. Sin renovaciones silenciosas con cambio de precio, sin cobros por funciones descritas como gratuitas.
5. Monetización del apuro. No cobraremos por desbloquear un recordatorio urgente ni por exportar los datos propios del usuario.

Si una oportunidad de ingreso choca con esta lista, se rechaza aunque sea rentable.

## 10. Riesgos del modelo

| Riesgo | Descripción | Mitigación |
| --- | --- | --- |
| Disposición a pagar baja | El usuario colombiano espera apps financieras gratuitas (Nequi, Daviplata, billeteras sin costo) y no paga [HIPÓTESIS] | Validar precio antes de construir Premium; mantener el Free útil aunque la conversión sea baja |
| Conversión menor a 5% | El rango 5-8% no se alcanza y el ingreso no cubre costos [HIPÓTESIS] | Revisar el límite gratuito y el paquete Premium; compuerta PIVOT del roadmap |
| CAC mayor a $50.000 COP | Adquirir usuarios de pago cuesta más que su valor temprano [HIPÓTESIS] | Crecer por recomendación y contenido; evitar pauta masiva hasta tener retención probada |
| Costos de IA superiores | El OCR real cuesta más de $0,10/usuario/mes por baja adopción del XML gratuito [ESTIMACIÓN] | Caché por CUFE y por proveedor más importe; procesamiento por lotes; umbral de confianza en 0,85 |
| Sensibilidad cultural al precio | $20.000 COP/mes compite con servicios esenciales y se percibe caro [HIPÓTESIS] | Plan anual con 2 meses gratis; comunicar el ahorro en multas evitadas (~$20.000 COP por evento) |
| Dependencia de un solo ingreso | Solo suscripción, sin diversificación inicial | Decisión consciente [DECISIÓN]; diversificar solo con funciones que respeten la lista de lo que NO haremos |

## 11. Validación y próximos pasos

Este modelo es un conjunto de hipótesis encadenadas, no un plan financiero cerrado. La validación sigue el plan de ../product/product-validation.md: entrevistas con guion de 18 preguntas, encuesta de 10 preguntas, experimento de página de aterrizaje, conserje manual con 20 usuarios y prueba de puerta falsa para el escaneo de facturas. Las compuertas GO, PIVOT y STOP de ese documento deciden si el modelo avanza, se ajusta o se abandona. Ningún precio se publica antes de superar esas compuertas [DECISIÓN].

## 12. Fuentes y convención de etiquetas

Convención usada en este documento: [DATO] es un hecho verificado con `Fuente:` y fecha de consulta; [HIPÓTESIS] es una suposición por validar; [ESTIMACIÓN] es un cálculo con metodología explícita; [DECISIÓN] es una elección de diseño del equipo. Precios de competidores y proveedores verificados el 07/10/2026 en finku.co, supabase.com/pricing, render.com/pricing, fly.io/pricing, cloudflare.com y aws.amazon.com. Las anclas de Spotify (~$18.900 COP/mes), Netflix (~$50.000 COP/mes) y multas (~$20.000 COP) son referencias de contexto marcadas [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.
