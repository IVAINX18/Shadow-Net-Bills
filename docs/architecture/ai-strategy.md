# Estrategia de IA y OCR (V1 y posterior) - Shadow-Net Bills

> Estado: estrategia para V1 y posterior. El MVP no incluye IA ni OCR. Todo el contenido de este documento describe trabajo futuro y no compromete fechas. Fecha de redacción: 07/10/2026.

## 1. Principio rector

La IA se usa solo donde aporta valor medible y no existe una alternativa determinista más barata, más rápida y más auditable. La regla de decisión es simple:

1. Si el problema se resuelve con reglas, consultas SQL o comparaciones directas, se resuelve sin IA.
2. Si el problema exige interpretar documentos no estructurados (fotos o PDF de facturas sin XML), se evalúa OCR en la nube.
3. Los modelos de lenguaje (LLM) son el último recurso, solo para los casos donde el OCR declara confianza baja.
4. Nada de IA en el MVP. El MVP es manual-primero: el usuario registra sus obligaciones a mano y el sistema solo almacena, calcula vencimientos y envía recordatorios con lógica determinista.

Esta postura mantiene los costos predecibles, reduce la superficie de privacidad y evita que un error del modelo se convierta en un dato financiero falso sin revisión humana.

## 2. Alcance por fases

| Fase | Contenido de IA/OCR | Estado |
| --- | --- | --- |
| MVP | Ninguno. CRUD de obligaciones, recurrencia determinista, recordatorios 3/1/0 días, resumen mensual por agregación SQL, exportación CSV | Alcance aprobado |
| V1 | Importación de facturas: parseo gratuito de XML DIAN + OCR en la nube para foto/PDF + clasificación por reglas | Estrategia (este documento) |
| V2 | Detección simple de anomalías con estadística descriptiva; resumen mensual asistido opcional con LLM; detección determinista de cambios de precio | Estrategia (este documento) |
| V3 y futuro | Integraciones de correo y calendario; Open Banking cuando sea operativo en Colombia (2027 o posterior) | Solo roadmap |

La clasificación de categorías, el cálculo de próximos vencimientos, la detección de cambios de precio y los recordatorios quedan implementados con lógica determinista en todas las fases. No se reimplementan con IA lo que ya funciona sin IA.

## 3. Evaluación por caso de uso

Cada caso de uso se evalúa con seis dimensiones: beneficio, costo, complejidad, riesgos, privacidad y si requiere IA.

| Caso de uso | Beneficio | Costo | Complejidad | Riesgos | Privacidad | Requiere IA? |
| --- | --- | --- | --- | --- | --- | --- |
| Extracción de facturas (OCR) | Alto: elimina el registro manual, acelera la adopción en V1 | Medio: unos $10 por cada 1.000 páginas con OCR preentrenado en la nube | Media: integrar proveedor, normalizar campos colombianos (NIT, CUFE, IVA, totales en COP) | Medio: errores de lectura en fotos borrosas; se mitiga con confirmación humana obligatoria | Alta sensibilidad: la factura contiene nombre, NIT, consumos y valores; exige consentimiento de transferencia internacional y DPA | Sí, en V1, solo para imagen/PDF sin XML |
| Clasificación por categoría | Medio: ahorra un clic al usuario | Cero marginal | Baja | Bajo: una categoría errada es visible y corregible | Nula exposición adicional: opera sobre datos ya almacenados | No. Decisión: reglas sin IA (mapeo proveedor a categoría + palabras clave + categoría por defecto con confirmación) |
| Detección de anomalías (cobros atípicos) | Medio: alerta sobre subidas inusuales | Cero marginal | Baja-media | Medio: falsos positivos si los umbrales son rígidos; se mitiga con umbrales por categoría configurables | Nula exposición adicional: cálculo local sobre el historial propio | No. Decisión: estadística simple en V2 (media móvil y desviación por categoría, sin modelos) |
| Detección de cambios de precio | Medio: muestra si un proveedor subió la tarifa mes a mes | Cero marginal | Baja | Bajo: comparación exacta entre periodos | Nula exposición adicional | No. Decisión: comparación determinista en V2 (mismo proveedor y concepto, diferencia de importes y porcentaje) |
| Resumen financiero mensual en lenguaje natural | Medio-bajo: texto legible del mes; el resumen numérico ya existe por SQL | Bajo: un LLM pequeño cuesta menos de $0,01 por resumen con pipeline escalonado | Media: plantilla con datos verificados + generación + revisión | Medio: el modelo podría inventar cifras; se mitiga pasando solo totales ya calculados y prohibiendo que genere números nuevos | Media: se envían solo agregados anonimizados, nunca documentos completos | Opcional. Decisión: LLM opcional en V2, desactivado por defecto, solo si el usuario lo activa |
| Búsqueda semántica ("factura cara de luz de marzo") | Bajo en esta etapa: el volumen de obligaciones del MVP no la justifica | Medio-alto: índice vectorial, embeddings y su operación continua | Alta | Medio: resultados aproximados difíciles de explicar | Media: duplica el contenido en un índice externo | No. Decisión: postergar más allá de V2; la búsqueda por texto exacto y filtros cubre la necesidad |

Conclusión de la tabla: el único caso que requiere IA en V1 es la extracción de facturas desde imagen o PDF sin XML. Todo lo demás se resuelve con reglas, SQL o estadística descriptiva.

## 4. Pipeline OCR escalonado

El pipeline ordena las rutas de más barata a más cara y se detiene en la primera que entrega un resultado confiable. El objetivo es que entre el 40 % y el 60 % de las facturas colombianas se resuelvan gratis mediante el XML de la DIAN.

```
Entrada: archivo subido por el usuario (XML, PDF o foto)
  |
  v
[1] Es XML de factura electrónica DIAN (UBL 2.1 con CUFE)?
  |  SI -> parseo local gratuito: NIT, emisor, CUFE, totales, IVA, fecha
  |         costo $0. Fin del pipeline.
  |  NO
  v
[2] OCR en la nube con modelo preentrenado de facturas
  |  Extrae: emisor, NIT, fecha, líneas, subtotal, IVA, total, moneda
  |  Cada campo llega con puntaje de confianza 0 a 1.
  v
[3] Confianza mínima >= 0,85 en los campos críticos (total, fecha, emisor)?
  |  SI -> prellenar el formulario de obligación para revisión del usuario.
  |  NO -> [4] reintento con LLM de bajo costo solo sobre los campos dudosos.
  v
[4] Escalamiento a LLM económico (solo campos de baja confianza)
  |  Entrada: recorte del documento + salida del OCR, sin datos de otras facturas.
  v
[5] Confirmación humana obligatoria
     El usuario revisa y corrige cada campo antes de guardar.
     Nada se guarda como definitivo sin esta confirmación.
     El documento original se conserva cifrado para auditoría de la extracción.
```

Reglas del pipeline:

- La confirmación humana es obligatoria en todos los caminos, incluso con confianza 1,0. Un valor mal leído que se convierte en obligación genera recordatorios falsos.
- El LLM nunca recibe la factura completa si el OCR ya extrajo parte con confianza alta; recibe solo los fragmentos dudosos (minimización de datos).
- El resultado del OCR y del LLM se registra con proveedor, fecha y puntajes para poder auditar errores.
- Los archivos maliciosos se rechazan antes del pipeline con validación de tipo MIME y bytes mágicos, límite de tamaño y antivirus del almacenamiento.

## 5. Comparativa de proveedores OCR

Precios verificados en las páginas de precios de cada proveedor, 2026. Los valores en dólares se mantienen en USD porque los proveedores facturan en esa moneda; los valores en COP de este documento usan formato $20.000.

| Proveedor | Modelo relevante | Precio aproximado | Nivel gratuito | Retención y borrado | Notas para Bills |
| --- | --- | --- | --- | --- | --- |
| Azure Document Intelligence (preentrenado de facturas) | prebuilt-invoice | Unos $10 por cada 1.000 páginas | 500 páginas por mes | Elimina la entrada después de 24 horas | Recomendado como primera opción: precisión en facturas latinas, borrado documentado, DPA disponible |
| AWS Textract (AnalyzeExpense) | AnalyzeExpense | Unos $10 por cada 1.000 páginas | Periodo de prueba de 3 meses | Sin retención de contenido por defecto | Recomendado como alternativa: paridad de precio con Azure, sin retención de contenido |
| Google Document AI (preentrenado) | Invoice Parser | Unos $10 por cada 1.000 páginas | 1.000 páginas por mes | Retención según configuración del proyecto | Viable: nivel gratuito generoso para pilotos; revisar configuración de región y retención |
| Mindee (API de facturas) | Invoice API | Plan por volumen, desde tramo gratuito limitado | Tramo gratuito limitado | Retención según plan | Viable para volúmenes bajos; evaluar costo por encima de 5.000 páginas mensuales |
| Rossum | Plataforma de documentos | Precio empresarial por cotización | Sin tramo gratuito permanente | Retención contractual | Descartado para V1: orientado a empresas grandes, sin precio público por página |

Recomendación: contratar Azure Document Intelligence o AWS Textract con modelo preentrenado de facturas, sin entrenar modelos propios. El volumen de V1 no justifica un modelo dedicado. La interfaz desacoplada (sección 10) permite cambiar de proveedor sin reescribir el producto.

Fuente: páginas de precios de Azure, AWS, Google Cloud, Mindee y Rossum, consultadas en 2026. Fuente: investigación interna Shadow-Net, 2026-10-07 para la síntesis comparativa.

## 6. Comparativa de modelos LLM de apoyo

El LLM solo interviene en el paso [4] del pipeline, para campos con confianza menor a 0,85. Costos por factura verificados en 2026:

| Modelo | Costo aproximado por factura | Rol en Bills |
| --- | --- | --- |
| GPT-4o-mini | Unos $0,00068 por factura | Primera opción para reintento de campos dudosos |
| Gemini Flash-Lite | Unos $0,00045 por factura | Alternativa más barata con calidad suficiente para normalizar fechas y valores |
| Claude Haiku | Unos $0,005 por factura | Reserva si se necesita mejor seguimiento de instrucciones en documentos difíciles |

El costo objetivo por factura extraída se mantiene por debajo de $0,01 combinando el parseo gratuito de XML DIAN (40 % a 60 % de los casos en Colombia) con OCR a unos $10 por cada 1.000 páginas y LLM solo en la minoría de baja confianza. Ningún resumen o análisis usa el modelo grande más caro; esas tareas usan agregación SQL o el modelo pequeño bajo plantilla cerrada.

## 7. Privacidad y protección de datos

Toda la estrategia de IA opera bajo la Ley 1581 de 2012. Los proveedores de OCR y LLM procesan fuera de Colombia, por lo que aplican las reglas de transferencia internacional.

1. Consentimiento explícito: antes del primer documento, el usuario acepta de forma previa, expresa e informada que su factura será procesada por un proveedor en la nube fuera del país. Sin ese consentimiento no se habilita la importación; el registro manual sigue disponible.
2. Acuerdo de tratamiento (DPA): cada proveedor de OCR o LLM debe firmar un acuerdo de encargado del tratamiento que limite el uso a la extracción solicitada, prohíba el entrenamiento con nuestros documentos y fije plazos de borrado. Sin DPA firmado no se contrata al proveedor.
3. Minimización: al OCR se envía solo el documento; al LLM solo los fragmentos de baja confianza. Nunca se envían el historial completo, otras facturas ni datos de la cuenta.
4. Borrado y retención: se aplica la ventana más corta disponible (referencia: Azure elimina la entrada después de 24 horas; AWS no retiene contenido). Los documentos del usuario en nuestro almacenamiento se borran cuando el usuario elimina su cuenta, junto con el resto de sus datos.
5. Derechos del titular: conocer, actualizar, rectificar y suprimir sus documentos y extracciones; revocar el consentimiento de procesamiento con IA en cualquier momento desde la configuración, conservando el uso manual.
6. Registro de incidentes: cualquier exposición a través de un proveedor sigue el procedimiento de respuesta a incidentes (notificación a la SIC dentro de los 15 días hábiles cuando aplique). El detalle operativo vive en docs/architecture/security.md y el marco legal en docs/legal/regulatory-compliance.md.

## 8. Costos proyectados

Supuestos de cálculo: cada usuario activo importa en promedio 10 facturas al mes; entre el 40 % y el 60 % llegan como XML DIAN gratuito; el resto pasa por OCR a unos $10 por cada 1.000 páginas; el LLM interviene en una minoría de casos a menos de $0,01 por factura.

| Escala | Facturas al mes | Costo mensual estimado de IA/OCR | Costo por usuario al mes |
| --- | --- | --- | --- |
| Piloto V1 (100 usuarios) | 1.000 | Unos $10 a $12 | Unos $0,10 a $0,12 |
| V1 (1.000 usuarios) | 10.000 | Unos $105 | Unos $0,10 |
| V2 (10.000 usuarios) | 100.000 | Unos $1.050 | Unos $0,10 |

Lectura ejecutiva: el costo de IA por usuario se mantiene cerca de $0,10 al mes a escala porque el XML gratuito absorbe la mitad del volumen y el OCR unitario es de un centavo por página. A 1.000 usuarios el gasto mensual ronda los $105, cifra que cabe dentro del margen del nivel Premium hipotético (hasta $20.000 COP por mes) incluso con una sola docena de suscriptores de pago. Los valores de negocio son hipótesis de trabajo, no compromisos de precio.

## 9. Optimización de costos

1. Caché por CUFE: el CUFE identifica de forma única cada factura electrónica colombiana. Si el CUFE ya fue procesado, se reutiliza el resultado y no se paga dos veces.
2. Caché por proveedor e importe: para facturas recurrentes idénticas (mismo NIT emisor, mismo total, mismo periodo), se sugiere la plantilla anterior sin llamar al proveedor.
3. Procesamiento por lotes (batching): las importaciones múltiples se agrupan y se envían fuera de picos, reduciendo reintentos y aprovechando ventanas de menor latencia.
4. Reducción de imagen (downscale): antes de enviar al OCR, la imagen se normaliza a la resolución mínima aceptada por el proveedor; menos píxeles significan menos páginas equivalentes y menos bytes transferidos.
5. Umbral de confianza calibrado: el 0,85 se revisa con datos reales del piloto; subirlo aumenta el uso de LLM y el costo, bajarlo aumenta errores visibles.
6. XML primero, siempre: el orden del pipeline no es configurable por el usuario final; el parseo gratuito de DIAN siempre se intenta antes de gastar en OCR.
7. Topes por usuario: cuota mensual de importaciones con IA por plan; superado el tope, el usuario puede seguir registrando manualmente sin límite.

## 10. Arquitectura desacoplada

La IA vive detrás de una interfaz abstracta para que el proveedor sea intercambiable y el producto funcione aunque se apague la IA. La interfaz canónica se llama DocumentProcessor y ya está reservada en docs/architecture/architecture.md.

```
Oblicación manual (MVP) ---> API ---> Supabase (PostgreSQL)
                                  |
V1: Subida de factura ---> DocumentProcessor (interfaz)
                                  |
              +-------------------+-------------------+
              |                   |                   |
     Proveedor A (Azure)  Proveedor B (AWS Textract)  Parseo XML DIAN local
              |                   |                   |  (costo $0)
              +-------------------+-------------------+
                                  |
                     Resultado normalizado + confianza
                                  |
                     Confirmación humana obligatoria
                                  |
                          Guardar obligación
```

Contrato mínimo de DocumentProcessor:

- Entrada: bytes del documento, tipo MIME verificado, identificador de usuario, CUFE si se conoce.
- Salida: campos normalizados (emisor, NIT, fecha, subtotal, IVA, total, moneda COP por defecto), puntaje de confianza por campo, proveedor usado, costo imputable.
- Errores tipados: documento ilegible, formato no soportado, proveedor no disponible, timeout. Cada error devuelve una acción sugerida (reintentar, registrar manual).
- Ningún componente del producto importa el SDK del proveedor directamente; solo habla con DocumentProcessor. Cambiar de Azure a AWS Textract es cambiar una implementación, no el producto.
- Si todos los proveedores fallan, el sistema degrada a registro manual con los campos vacíos y un mensaje claro. La IA nunca bloquea el uso básico.

## 11. Límites explícitos y lo que este producto no hace

- El producto no ofrece asesoría financiera automática. No recomienda inversiones, no califica créditos, no sugiere endeudarse y no presenta ningún texto generado como recomendación profesional. Los resúmenes describen el pasado ("en septiembre pagó $320.000 en Servicios públicos") y nunca prescriben el futuro.
- La IA no decide vencimientos, no marca facturas como pagadas y no envía recordatorios por sí sola; esas acciones las ejecuta el usuario o el programador determinista con reglas auditables.
- No se construye puntaje crediticio paralelo ni se conserva información negativa más allá de lo necesario para el historial del propio usuario.
- No se almacenan credenciales bancarias, números completos de tarjetas ni contraseñas de terceros en ninguna fase.
- Cualquier ampliación de la IA (nuevos casos de uso, nuevos proveedores, activación del resumen con LLM) requiere actualizar este documento, la evaluación de privacidad y el consentimiento informado antes de implementarse.
