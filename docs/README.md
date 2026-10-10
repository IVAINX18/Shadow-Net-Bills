# Índice de documentación de Shadow-Net Bills

## Propósito

Este índice es la puerta de entrada a la especificación de Shadow-Net Bills. Enumera todos los documentos del proyecto, explica a quién va dirigido cada uno y propone rutas de lectura según el perfil del lector. No contiene decisiones nuevas de producto: solo organiza y enlaza la documentación existente.

## Estado del proyecto

El proyecto está en fase de planificación. Fecha de este índice: 07/10/2026.

No hay código todavía: el repositorio contiene únicamente la especificación en `docs/`. El desarrollo del MVP comenzará cuando se apruebe la documentación. El documento maestro que sintetiza todo es [SHADOW_NET_BILLS_SPECIFICATION.md](./SHADOW_NET_BILLS_SPECIFICATION.md).

## Tabla de documentos

| Documento | Descripción | Ruta relativa | Para quién |
| --- | --- | --- | --- |
| Visión y propuesta de valor | Visión, problema, solución y propuesta de valor seleccionada | [./product/product-overview.md](./product/product-overview.md) | fundador |
| Análisis competitivo | Comparativa de 10 competidores y contexto del mercado colombiano | [./product/competitive-analysis.md](./product/competitive-analysis.md) | fundador |
| Requisitos | Requisitos funcionales y no funcionales clasificados con MoSCoW | [./product/requirements.md](./product/requirements.md) | ingeniero |
| Casos de uso | 16 casos de uso del MVP y de fases posteriores | [./product/use-cases.md](./product/use-cases.md) | ingeniero |
| Roadmap | Fases 0 a 7 con compuertas de validación GO/PIVOT/STOP | [./product/roadmap.md](./product/roadmap.md) | fundador |
| Validación de producto | Hipótesis, entrevistas, encuesta y experimentos previos al desarrollo | [./product/product-validation.md](./product/product-validation.md) | fundador |
| Arquitectura | Componentes, flujos y decisiones de arquitectura del monolito | [./architecture/architecture.md](./architecture/architecture.md) | ingeniero |
| Modelo de datos | 6 tablas del MVP con DDL, índices y políticas RLS | [./architecture/database.md](./architecture/database.md) | ingeniero |
| API REST | Convenciones y endpoints del MVP con esquemas JSON | [./architecture/api.md](./architecture/api.md) | ingeniero |
| Seguridad | Modelo de amenazas, controles y respuesta a incidentes | [./architecture/security.md](./architecture/security.md) | ingeniero |
| Estrategia de IA | Evaluación de IA por caso de uso y pipeline de OCR para V1 | [./architecture/ai-strategy.md](./architecture/ai-strategy.md) | ingeniero |
| Flujos de usuario | 15 pantallas y flujos principales de la aplicación | [./ux/user-flows.md](./ux/user-flows.md) | ingeniero |
| Modelo de negocio | Tiers Free y Premium, costos y unit economics como hipótesis | [./business/business-model.md](./business/business-model.md) | negocio |
| Análisis de mercado | Datos, hipótesis y estimaciones TAM/SAM/SOM del mercado colombiano | [./business/market-analysis.md](./business/market-analysis.md) | negocio |
| Cumplimiento regulatorio | Consideraciones legales colombianas con aviso de no asesoría | [./legal/regulatory-compliance.md](./legal/regulatory-compliance.md) | legal |
| Selección de stack | Decisiones tecnológicas por componente con alternativas y costos | [./technical/stack-selection.md](./technical/stack-selection.md) | ingeniero |
| Infraestructura | Comparativa de proveedores, costos y estrategia de despliegue | [./technical/infrastructure.md](./technical/infrastructure.md) | ingeniero |
| Estructura de repositorio | Estructura de carpetas propuesta para la fase de desarrollo | [./technical/repository-structure.md](./technical/repository-structure.md) | ingeniero |
| Especificación maestra | Síntesis de las 20 secciones con enlaces a cada documento | [./SHADOW_NET_BILLS_SPECIFICATION.md](./SHADOW_NET_BILLS_SPECIFICATION.md) | fundador |
| Índice de documentación | Este índice: propósito, tabla de documentos y rutas de lectura | [./README.md](./README.md) | fundador |

## Rutas de lectura recomendadas

### Nuevo en el proyecto

1. Este índice para orientarse.
2. La [especificación maestra](./SHADOW_NET_BILLS_SPECIFICATION.md) para la visión completa en 20 secciones.
3. [Visión y propuesta de valor](./product/product-overview.md) para el porqué del producto.
4. [Arquitectura](./architecture/architecture.md) para el cómo técnico.

### Fundador evaluando el proyecto

1. [Validación de producto](./product/product-validation.md) para las hipótesis y cómo comprobarlas.
2. [Modelo de negocio](./business/business-model.md) para tiers, costos y unit economics.
3. [Roadmap](./product/roadmap.md) para fases y compuertas GO/PIVOT/STOP.
4. [Análisis de mercado](./business/market-analysis.md) para datos frente a hipótesis.

### Ingeniero implementando el MVP

1. [Arquitectura](./architecture/architecture.md) para componentes y flujos.
2. [Modelo de datos](./architecture/database.md) para tablas, DDL y políticas RLS.
3. [API REST](./architecture/api.md) para endpoints y esquemas JSON.
4. [Seguridad](./architecture/security.md) para amenazas, controles y checklist pre-lanzamiento.
5. [Requisitos](./product/requirements.md) y [casos de uso](./product/use-cases.md) para el detalle funcional.

## Cómo mantener la documentación

- Cada documento tiene un único responsable de edición: se edita su propio archivo y nunca el de otro documento.
- Los enlaces son relativos al archivo que los contiene: desde `docs/` se usa `./product/...`, `./architecture/...` o `./SHADOW_NET_BILLS_SPECIFICATION.md`.
- Tras cualquier cambio, ejecutar el bucle canónico de verificación de enlaces sobre `docs/` y confirmar que imprime LINKS OK.
- Mantener un solo `#` H1 por archivo, secciones con `##`, tablas bien formadas, fechas en formato DD/MM/AAAA y valores en formato $20.000.
- No usar emojis ni marcadores de pendiente en ningún documento.
- Etiquetar cada dato de mercado, precio o afirmación legal con `[DATO]`, `[HIPÓTESIS]`, `[ESTIMACIÓN]` o `[DECISIÓN]`, con `Fuente:` cuando corresponda.
- Este índice se actualiza solo si se añade, elimina o renombra un documento: no duplica contenido de producto.

## Glosario

- **Obligación:** factura o pago recurrente que el usuario registra para no olvidar su vencimiento.
- **Recurrencia:** repetición programada de una obligación (única, semanal, mensual, trimestral o anual).
- **Cuota:** cada pago parcial de una compra a plazos registrada como obligación de categoría Financiero.
- **Recordatorio:** aviso por correo enviado 3, 1 y 0 días antes del vencimiento de una obligación.
- **RLS:** seguridad a nivel de fila (Row Level Security) que limita cada consulta a los datos del propio usuario.
- **MVP:** producto mínimo viable: versión manual-primero de 8 semanas sin IA, sin OCR y sin integraciones bancarias.
