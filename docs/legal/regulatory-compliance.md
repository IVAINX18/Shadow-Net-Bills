> Este documento es investigación informativa y NO constituye asesoramiento jurídico. Consulte a un abogado colombiano antes de tomar decisiones de cumplimiento.

# Consideraciones regulatorias de Shadow-Net Bills en Colombia

## 1. Resumen ejecutivo

Shadow-Net Bills es una aplicación de organización de obligaciones financieras personales que opera con un modelo manual-primero: el usuario registra sus facturas y pagos, y la aplicación envía recordatorios por correo electrónico [DECISIÓN]. La aplicación NO mueve dinero, NO almacena credenciales bancarias ni números completos de tarjetas, NO inicia pagos y NO se presenta como entidad financiera [DECISIÓN]. Este diseño reduce la superficie regulatoria, pero NO elimina las obligaciones de protección de datos personales ni las obligaciones tributarias propias del negocio [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

Los tres frentes regulatorios relevantes son: (a) la Ley 1581 de 2012 de protección de datos personales, vigilada por la Superintendencia de Industria y Comercio (SIC); (b) el estado del Open Banking en Colombia, que para el MVP es una dependencia futura y NO una vía de integración disponible; y (c) las obligaciones ante la DIAN como prestador de un servicio de software por suscripción [DATO]. Fuente: sic.gov.co; superfinanciera.gov.co; dian.gov.co, consultado 2026-10-07.

Este documento NO afirma cumplimiento definitivo de ninguna norma. Cada afirmación legal lleva etiqueta [DATO], [HIPÓTESIS], [ESTIMACIÓN] o [DECISIÓN] según la convención del proyecto, y las reformas en trámite se identifican como tales. Antes de lanzar, el equipo debe validar todo el contenido con un abogado colombiano [DECISIÓN].

## 2. Alcance y convenciones de este documento

Este documento cubre únicamente el marco colombiano aplicable al MVP descrito en la especificación: registro manual de obligaciones, recordatorios por correo, calendario, resumen mensual, exportación CSV y borrado de cuenta [DECISIÓN]. No cubre la regulación de otros países, la regulación financiera de captación o crédito, ni el régimen de telecomunicaciones [DECISIÓN].

Convenciones usadas en este documento: [DATO] indica un hecho verificado con su fuente citada; [HIPÓTESIS] indica una suposición del equipo que debe validarse; [ESTIMACIÓN] indica una cifra calculada con metodología explícita; [DECISIÓN] indica una decisión de diseño del proyecto [DECISIÓN]. Las citas siguen el formato `Fuente: <nombre o URL>, consultado 2026-10-07`. Cuando no existe una URL pública para el dato, se usa `Fuente: investigación interna Shadow-Net, 2026-10-07`.

## 3. Ley 1581 de 2012: principios y ámbito de aplicación

La Ley 1581 de 2012 es el régimen general de protección de datos personales en Colombia y aplica a Shadow-Net Bills desde el primer usuario, porque la aplicación trata datos personales como nombres, correos electrónicos, montos de obligaciones y fechas de vencimiento [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

Los principios rectores que el diseño debe respetar son: legalidad, finalidad, libertad, veracidad o calidad, transparencia, acceso y circulación restringida, seguridad y confidencialidad [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. En la práctica esto significa: recoger solo los datos necesarios para la funcionalidad ofrecida (minimización), usarlos únicamente para la finalidad informada al titular (limitación de la finalidad), mantenerlos exactos y actualizados, protegerlos con medidas técnicas y administrativas, y no comunicarlos a terceros sin autorización [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

La calidad de responsable del tratamiento recae en quien decide sobre las bases de datos y su finalidad, y la de encargado en quien las trata por cuenta del responsable; los proveedores de infraestructura (por ejemplo, alojamiento de base de datos y envío de correo) actúan típicamente como encargados o transmisores y deben quedar cubiertos por contratos de transmisión de datos [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

## 4. Autorización previa, expresa e informada y derechos del titular

Todo tratamiento de datos personales requiere la autorización previa, expresa e informada del titular, que debe obtenerse por medios que permitan su consulta posterior [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. Para Bills esto implica un flujo de registro donde el usuario acepta de forma separada los términos del servicio y la política de tratamiento de datos, con casillas independientes para finalidades opcionales como comunicaciones de mercadeo (opt-in) [DECISIÓN].

Las autorizaciones deben ser granulares y versionadas: el sistema debe registrar qué versión de la política aceptó cada usuario y cuándo, para poder demostrar la autorización ante la SIC si es requerida [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

Los derechos del titular comprenden conocer, actualizar y rectificar sus datos, solicitar prueba de la autorización, revocar la autorización, solicitar la supresión cuando el tratamiento no respete los principios, y presentar quejas ante la SIC [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. El MVP debe implementar estos derechos mediante las funciones de configuración, edición de obligaciones, exportación CSV y borrado de cuenta, con un procedimiento interno que responda consultas en un máximo de 10 días hábiles y reclamos en un máximo de 15 días hábiles [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

## 5. Transferencia internacional de datos

La infraestructura prevista (base de datos en la nube, envío de correo transaccional y futuros proveedores de OCR e IA) procesa datos fuera de Colombia, lo que constituye transferencia internacional de datos [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. La transferencia internacional exige autorización del titular y un nivel adecuado de protección en el país receptor [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

Los Estados Unidos figuran en la lista de países con nivel adecuado de protección según la actualización del Título V impulsada por la SIC, estado a 2026 [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. Dado que los proveedores previstos operan bajo jurisdicción estadounidense, esta adecuación simplifica el análisis, pero NO sustituye la autorización del titular ni los contratos de transmisión con cada proveedor [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. Para la fase V1 con OCR e IA, la política debe incluir un consentimiento explícito y separado para la transferencia internacional, además de acuerdos de tratamiento de datos (DPA) con cada proveedor y verificación de sus políticas de retención (por ejemplo, borrado de insumos tras el procesamiento) [DECISIÓN].

## 6. Registro Nacional de Bases de Datos (RNBD)

El Registro Nacional de Bases de Datos (RNBD), administrado por la SIC, exige el registro de las bases de datos a los responsables cuyo patrimonio o activos superen los 100.000 UVT, según los umbrales vigentes de la regulación [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. Una startup en etapa inicial probablemente queda exenta de la obligación de registro al no alcanzar ese umbral, pero todos los demás deberes de la Ley 1581 de 2012 aplican desde el primer día [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

El equipo debe monitorear el crecimiento de los activos y registrar las bases de datos en el RNBD en cuanto se supere el umbral aplicable, y en todo caso mantener el inventario interno de bases de datos, finalidades y encargados desde el MVP [DECISIÓN]. La exención del registro NO es una exención del cumplimiento: principios, autorizaciones, derechos del titular, seguridad y reporte de incidentes aplican con independencia del registro [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

## 7. Incidentes de seguridad y notificación a la SIC

Ante un incidente que comprometa datos personales, el responsable debe notificar a la SIC a través del RNBD dentro de los 15 días hábiles siguientes al conocimiento del incidente [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. El documento de seguridad del proyecto define el flujo de respuesta (detección, contención, evaluación, notificación y post-mortem) y este documento lo adopta como referencia operativa; ver `../architecture/security.md` [DECISIÓN].

El playbook de brechas del MVP debe incluir como mínimo: responsable de declarar el incidente, criterios de severidad, plantilla de notificación a la SIC, procedimiento de comunicación a los titulares afectados cuando proceda, y registro de lecciones aprendidas [DECISIÓN]. Los registros de aplicación nunca deben contener datos personales sensibles ni secretos, para que los propios logs no se conviertan en una segunda brecha [DECISIÓN].

## 8. Régimen sancionatorio y actividad reciente de la SIC

El incumplimiento de la Ley 1581 de 2012 puede acarrear sanciones de hasta 2.000 salarios mínimos legales mensuales vigentes (SMLMV), además de órdenes de corrección, bloqueo temporal y cierre de operaciones en casos graves [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

La actividad de enforcement muestra una SIC activa: durante 2025 se adelantaron 101 investigaciones en materia de protección de datos, con multas por un valor aproximado de $5.157 millones [DATO]. Fuente: sic.gov.co, consultado 2026-10-07. En marzo de 2025 la SIC sancionó a Mercado Libre con una multa aproximada de $214 millones por condicionar el acceso a un servicio al tratamiento de datos biométricos [DATO]. Fuente: sic.gov.co, consultado 2026-10-07.

La lección operativa para Bills es directa: nunca condicionar una funcionalidad esencial a la entrega de datos innecesarios, y mantener cada finalidad atada a su propia autorización [DECISIÓN]. El MVP, al no usar biometría ni datos sensibles, tiene un perfil de riesgo menor que los casos sancionados, pero el volumen de multas confirma que la SIC ejerce vigilancia efectiva y que el cumplimiento no es opcional [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07.

## 9. Habeas Data financiero: Ley 1266 de 2008

La Ley 1266 de 2008 (Habeas Data financiero) regula a los operadores de información crediticia y a las fuentes que reportan comportamiento financiero [DATO]. Fuente: funcionpublica.gov.co, consultado 2026-10-07. Shadow-Net Bills NO es un operador de información crediticia: no reporta a centrales de riesgo, no calcula puntajes de crédito y no comparte historiales de pago con terceros [DECISIÓN].

En consecuencia, el régimen de la Ley 1266 de 2008 no le aplica como operador, pero el equipo debe evitar dos conductas prohibidas en espíritu y letra: conservar información negativa desactualizada de los usuarios como si fuera un historial crediticio paralelo, y construir cualquier forma de calificación crediticia propia a partir de los pagos registrados [DATO]. Fuente: funcionpublica.gov.co, consultado 2026-10-07. Los datos de pagos existen únicamente para mostrar el historial personal del usuario y generar sus recordatorios, y se eliminan con el borrado de cuenta [DECISIÓN].

## 10. Open Banking y finanzas abiertas en Colombia

El Decreto 0368 de 2026, expedido en abril de 2026, adoptó el marco de finanzas abiertas obligatorio para las entidades vigiladas por la Superintendencia Financiera de Colombia (SFC) [DATO]. Fuente: minhacienda.gov.co, consultado 2026-10-07. Los terceros no vigilados pueden vincularse de forma voluntaria como Tercero Receptor No Vigilado, conforme a la Circular Externa 006 de 2025 de la SFC [DATO]. Fuente: superfinanciera.gov.co, consultado 2026-10-07.

El cronograma de estándares técnicos de la SFC se espera durante el segundo semestre de 2026, por lo que el uso operativo realista de las interfaces de finanzas abiertas se proyecta a partir de 2027 [DATO]. Fuente: superfinanciera.gov.co, consultado 2026-10-07. Para el MVP esto significa que NO existe integración bancaria disponible ni exigible: el registro manual es la única vía, por decisión de diseño y por realidad regulatoria [DECISIÓN].

Queda PROHIBIDO el scraping bancario: la aplicación nunca solicitará ni almacenará credenciales de banca en línea de los usuarios, ni usará pantallas automatizadas para extraer movimientos [DECISIÓN]. Cualquier integración futura con finanzas abiertas se hará únicamente a través de las interfaces oficiales, con autorización expresa del usuario y tras la evaluación jurídica correspondiente [DECISIÓN].

## 11. Obligaciones ante la DIAN

Como prestador de un servicio de software por suscripción, Shadow-Net Bills debe emitir su propia facturación electrónica conforme a los requisitos de la DIAN, usando el formato XML UBL 2.1 con su código CUFE [DATO]. Fuente: dian.gov.co, consultado 2026-10-07. La facturación electrónica del negocio es independiente del contenido que los usuarios registren en la aplicación: una cosa es facturar el servicio y otra es leer las facturas del usuario (esto último corresponde a la fase V1 con OCR) [DECISIÓN].

Los documentos tributarios y contables deben conservarse entre 5 y 10 años según el tipo documental, por lo que el equipo debe definir una tabla de retención desde el inicio, aun cuando el MVP no procese facturas de los usuarios [DATO]. Fuente: dian.gov.co, consultado 2026-10-07. El parseo gratuito de XML de facturas electrónicas (formato UBL 2.1) es la vía prevista para la fase V1 de importación, porque evita costos de OCR en el subconjunto de facturas electrónicas colombianas [ESTIMACIÓN]. Fuente: investigación interna Shadow-Net, 2026-10-07.

| Documento | Retención recomendada | Base |
| --- | --- | --- |
| Facturas electrónicas propias emitidas | 10 años | Normativa tributaria [DATO]. Fuente: dian.gov.co, consultado 2026-10-07 |
| Soportes contables del negocio | 10 años | Normativa contable y tributaria [DATO]. Fuente: dian.gov.co, consultado 2026-10-07 |
| Contratos con proveedores | 5 años tras su terminación | Política interna propuesta [DECISIÓN] |
| Autorizaciones de tratamiento de datos | Vigencia del tratamiento más 4 años | Defensa ante eventuales reclamaciones [HIPÓTESIS]. Fuente: investigación interna Shadow-Net, 2026-10-07 |
| Registros de incidentes | 5 años | Trazabilidad de la respuesta [DECISIÓN] |

## 12. Checklist operativo de cumplimiento para el MVP

| Nro | Acción | Estado exigible al lanzar el MVP |
| --- | --- | --- |
| 1 | Publicar la política de tratamiento de datos con finalidades, derechos y canales de contacto | Obligatorio [DECISIÓN] |
| 2 | Implementar el aviso de privacidad en el registro y las casillas de autorización separadas (servicio, datos, mercadeo opt-in, transferencia internacional cuando aplique) | Obligatorio [DECISIÓN] |
| 3 | Versionar las autorizaciones y registrar qué versión aceptó cada usuario | Obligatorio [DECISIÓN] |
| 4 | Firmar contratos de transmisión de datos con Supabase, el proveedor de correo y cualquier otro encargado | Obligatorio [DECISIÓN] |
| 5 | Definir el procedimiento de derechos del titular con SLA de 10 días hábiles para consultas y 15 días hábiles para reclamos | Obligatorio [DECISIÓN] |
| 6 | Publicar la tabla de retención documental y configurarla en los procesos de borrado | Obligatorio [DECISIÓN] |
| 7 | Aprobar el playbook de brechas con notificación a la SIC dentro de los 15 días hábiles | Obligatorio [DECISIÓN] |
| 8 | Realizar una evaluación de impacto (DPIA) antes de activar cualquier tratamiento con IA u OCR en V1 | Exigible en V1, planificar desde el MVP [DECISIÓN] |
| 9 | Verificar el umbral del RNBD cada trimestre y registrar las bases de datos si se superan los 100.000 UVT | Seguimiento trimestral [DECISIÓN] |
| 10 | Revisar con un abogado colombiano toda la documentación antes del lanzamiento público | Obligatorio [DECISIÓN] |

## 13. Incertidumbres y dependencias normativas

El Proyecto de Ley Estatutaria 282 de 2026 en la Cámara (P.L.E. 282-2026SC), que entre otros puntos clasificaría los datos biométricos como sensibles y haría obligatorias las evaluaciones de impacto y la figura del oficial de protección de datos en ciertos casos, se encuentra EN TRÁMITE legislativo y NO está vigente [DATO]. Fuente: camara.gov.co, consultado 2026-10-07. Ninguna decisión del MVP puede presentarlo como ley aplicable; el equipo debe seguir su trámite y preparar el DPIA y la gobernanza de datos de V1 como si fuera a exigirse [DECISIÓN].

El cronograma operativo del Open Banking también es una dependencia futura: los estándares de la SFC previstos para el segundo semestre de 2026 y la operación realista desde 2027 condicionan cualquier integración bancaria posterior al MVP [DATO]. Fuente: superfinanciera.gov.co, consultado 2026-10-07. La lista de adecuación de transferencia internacional y los umbrales del RNBD pueden actualizarse; el equipo debe revalidar estas referencias antes del lanzamiento y al menos una vez al año [DECISIÓN].

## 14. Fuentes oficiales

| Tema | Fuente oficial |
| --- | --- |
| Ley 1581 de 2012, RNBD, incidentes, sanciones y actividad de enforcement | sic.gov.co, consultado 2026-10-07 |
| Habeas Data financiero, Ley 1266 de 2008 | funcionpublica.gov.co, consultado 2026-10-07 |
| Decreto 0368 de 2026 de finanzas abiertas | minhacienda.gov.co, consultado 2026-10-07 |
| Circular Externa 006 de 2025 y cronograma de estándares | superfinanciera.gov.co, consultado 2026-10-07 |
| P.L.E. 282-2026SC, trámite legislativo | camara.gov.co, consultado 2026-10-07 |
| Facturación electrónica UBL 2.1, CUFE y retención documental | dian.gov.co, consultado 2026-10-07 |
| Supuestos de diseño y cifras no verificadas en fuente pública | Fuente: investigación interna Shadow-Net, 2026-10-07 |

Ninguna sección de este documento constituye asesoramiento jurídico ni declara el cumplimiento definitivo de Shadow-Net Bills con las normas citadas. El paso siguiente exigible es la revisión por un abogado colombiano antes del lanzamiento público [DECISIÓN].
