# Flujos de usuario y pantallas de Shadow-Net Bills

Este documento describe los principios de experiencia de usuario, los flujos principales y las 15 pantallas de Shadow-Net Bills: 11 del MVP con detalle completo y 4 de fases posteriores con diseño conceptual breve. Todo el contenido aplica al alcance aprobado: registro manual de obligaciones, sin movimiento de dinero, sin integraciones bancarias y sin IA ni OCR en el MVP. Las pantallas 12 a 15 pertenecen a V1 o V2 y se marcan como tales en cada caso.

## 1. Principios UX

1. **Manual primero, fricción mínima.** Registrar una obligación debe tomar menos de un minuto: nombre, valor en COP, categoría, fecha de vencimiento y frecuencia. Los campos opcionales (método de pago genérico, notas, recordatorios) nunca bloquean el guardado.
2. **Claridad sobre el próximo vencimiento.** La tabla `obligations` ordena la experiencia alrededor de la columna `next_due_date`: el usuario siempre sabe qué vence primero y cuánto falta. Los montos se muestran en formato COP como $20.000.
3. **Recordatorios predecibles.** Los avisos por correo se envían con offsets fijos de {3,1,0} días antes del vencimiento. La configuración permite activarlos o desactivarlos por obligación, sin sorpresas ni lógica oculta.
4. **Categorías fijas en español colombiano.** Las 9 categorías son siempre las mismas: Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno y Otros. No se crean categorías personalizadas en el MVP, lo que mantiene simples los filtros, el calendario y el resumen mensual.
5. **Lenguaje honesto sobre el alcance.** La interfaz nunca promete mover dinero, pagar facturas automáticamente ni conectarse con bancos. El método de pago registrado (tarjeta, transferencia, efectivo, débito automático, otro) es solo una nota informativa de cómo el usuario suele pagar.
6. **Los estados vacíos enseñan.** Cada lista vacía explica qué hacer a continuación con un texto concreto y una acción directa, en lugar de mostrar una pantalla en blanco.
7. **Móvil primero.** La mayoría de usuarios registrará y consultará obligaciones desde el teléfono. Cada pantalla del MVP define su comportamiento responsive/mobile de forma explícita.
8. **Errores accionables y estados de carga visibles.** Todo error explica qué pasó y cómo reintentarlo; toda espera mayor a un instante muestra un indicador de loading.

## 2. Flujos principales

**Flujo A. Registro, onboarding y primera obligación.** El visitante crea su cuenta con correo y contraseña, confirma el acceso, pasa por un onboarding de tres pasos (explicación del producto, selección de categorías de interés y activación de recordatorios) y registra su primera obligación. La meta de activación es que la primera obligación quede creada dentro de las 24 horas siguientes al registro. Si el usuario abandona el onboarding, al volver lo retoma donde quedó y el dashboard muestra un recordatorio amable hasta completar la primera obligación.

**Flujo B. Marcar una obligación como pagada.** Desde el dashboard, la lista de obligaciones o el detalle, el usuario pulsa "Marcar como pagada", confirma el pago (con fecha de pago por defecto hoy y método opcional) y el sistema registra el pago en el historial. Si la obligación es recurrente (semanal, mensual, trimestral o anual), se genera automáticamente el siguiente vencimiento actualizando `next_due_date`. El usuario ve una confirmación y el historial refleja el pago con su fecha.

**Flujo C. Configurar un recordatorio.** En el detalle o en la creación de una obligación, el usuario activa los recordatorios y elige entre los offsets {3,1,0} días. Guarda y recibe un mensaje de confirmación que indica cuántos avisos llegarán por correo antes del vencimiento. Puede desactivarlos en cualquier momento desde el mismo interruptor.

**Flujo D. Consultar el calendario.** El usuario abre el calendario mensual, navega entre meses, ve cada obligación ubicada en su fecha de vencimiento y pulsa un día para ver el detalle de lo que vence ese día. Los días con vencimientos próximos se distinguen visualmente de los vencidos y de los ya pagados.

**Flujo E. Exportar datos.** Desde la configuración, el usuario solicita la exportación en CSV de sus obligaciones y pagos. El sistema genera el archivo y lo ofrece para descarga. Si no hay datos, se muestra un estado vacío que explica que primero debe registrar obligaciones.

## 3. Pantallas del MVP y de fases posteriores

### 1. Login

**Propósito:** permitir que el usuario registrado inicie sesión y que el visitante nuevo llegue al registro o a la recuperación de contraseña.

**Información mostrada:** formulario con campos de correo electrónico y contraseña, enlace a "Crear cuenta", enlace a "Olvidé mi contraseña" y mensaje de propuesta de valor breve ("Nunca más olvides pagar una factura").

**Acciones:** iniciar sesión, ir al registro, solicitar recuperación de contraseña, mostrar u ocultar la contraseña.

**Estados vacíos:** no aplica como lista; si el usuario llega sin cuenta, el enlace de registro explica: "Si todavía no tienes cuenta, créala gratis en menos de un minuto".

**Errores:** credenciales inválidas ("El correo o la contraseña no coinciden. Revisa los datos e inténtalo de nuevo"); cuenta inexistente con sugerencia de registro; demasiados intentos con espera antes de reintentar; error de red con botón de reintentar.

**Loading:** el botón de acceso muestra indicador de progreso y se deshabilita mientras se valida la sesión.

**Responsive/mobile:** formulario centrado a una columna en móvil, teclado apropiado por campo (email y texto seguro), botón de acción a ancho completo y enlaces con área táctil amplia.

### 2. Onboarding

**Propósito:** guiar al usuario nuevo en tres pasos cortos hasta registrar su primera obligación dentro de las 24 horas siguientes al registro.

**Información mostrada:** paso 1, explicación del producto en una frase más tres beneficios (registra, recuerda con avisos a {3,1,0} días y consulta el calendario); paso 2, las 9 categorías fijas para marcar las de interés (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros); paso 3, activación de recordatorios por correo y botón para crear la primera obligación.

**Acciones:** avanzar y retroceder entre pasos, omitir el recorrido, seleccionar categorías de interés, activar o desactivar recordatorios, crear la primera obligación.

**Estados vacíos:** el estado vacío de esta pantalla es su razón de ser y usa el texto: "Todavía no tienes obligaciones registradas. Crea la primera ahora y activa tus recordatorios para no olvidarla". Si el usuario omite el recorrido, el dashboard conserva ese mismo mensaje hasta que registre algo.

**Errores:** si falla el guardado de preferencias, se muestra "No pudimos guardar tus preferencias. Inténtalo de nuevo" y el usuario puede reintentar sin perder el paso actual; error de red con reintento.

**Loading:** transición breve entre pasos e indicador al guardar preferencias antes de avanzar.

**Responsive/mobile:** pasos a pantalla completa en móvil con una sola idea por vista, botones grandes de avanzar y retroceder, y lista de categorías en tarjetas táctiles de dos columnas.

### 3. Dashboard

**Propósito:** responder de un vistazo qué vence pronto, qué está vencido y cuál es el total del mes en COP.

**Información mostrada:** tarjetas de resumen (total a pagar del mes, obligaciones vencidas, próximos vencimientos a 7 días), lista de próximas 5 obligaciones ordenadas por `next_due_date` con nombre, categoría, valor y días restantes, y accesos directos a crear obligación, calendario y resumen mensual.

**Acciones:** crear obligación, marcar como pagada desde la tarjeta, ir al detalle, abrir calendario, abrir resumen mensual, filtrar por categoría desde el resumen.

**Estados vacíos:** cuando no hay obligaciones, muestra el texto: "No tienes obligaciones por aquí. Registra tu primera factura, arriendo o suscripción para ver tu resumen del mes". Incluye botón directo a crear obligación. Si hay obligaciones pero ninguna vence este mes, indica: "Nada vence este mes. Todo al día por ahora".

**Errores:** si falla la carga del resumen, "No pudimos cargar tu resumen. Revisa tu conexión e inténtalo de nuevo" con botón de reintentar; los datos parciales se marcan como desactualizados en lugar de ocultarse.

**Loading:** tarjetas con esqueletos de carga y lista con marcadores mientras se consulta la API base `/api/v1/obligations`.

**Responsive/mobile:** tarjetas apiladas en una columna, lista compacta con valor y fecha visibles sin abrir el detalle, y botón flotante para crear obligación.

### 4. Obligaciones (lista)

**Propósito:** listar todas las obligaciones del usuario con filtros por estado y categoría para encontrar y gestionar cualquiera rápidamente.

**Información mostrada:** cada fila con nombre, valor en COP, categoría (una de las 9 fijas), fecha de vencimiento y `next_due_date` en recurrentes, estado (pendiente, pagada, vencida) y método de pago genérico como etiqueta informativa (tarjeta, transferencia, efectivo, débito automático, otro).

**Acciones:** filtrar por estado y por categoría, ordenar por vencimiento o valor, buscar por nombre, crear obligación, abrir detalle, marcar como pagada sin abrir el detalle, eliminar con confirmación.

**Estados vacíos:** sin obligaciones registradas, el estado vacío usa el texto: "Aún no registras ninguna obligación. Agrega tu primera factura para empezar a recibir recordatorios". Sin resultados para un filtro: "Ninguna obligación coincide con ese filtro. Ajusta la búsqueda o limpia los filtros para ver todo de nuevo".

**Errores:** error de carga con reintento; error al marcar como pagada con reversión optimista ("No se pudo marcar como pagada. Inténtalo de nuevo"); error al eliminar con confirmación de que nada se borró.

**Loading:** esqueleto de filas al cargar, indicador en línea al aplicar filtros y confirmación inmediata al marcar como pagada.

**Responsive/mobile:** filas en tarjetas de una columna, filtros en un panel plegable, acciones principales (pagar, ver) siempre visibles y acciones secundarias (editar, eliminar) en menú.

### 5. Detalle de obligación

**Propósito:** mostrar toda la información de una obligación y su historial de pagos en un solo lugar.

**Información mostrada:** nombre, valor, categoría, frecuencia (única, semanal, mensual, trimestral o anual), fecha de vencimiento, `next_due_date` si es recurrente, método de pago genérico, notas, estado de recordatorios con offsets {3,1,0} e historial de pagos con fechas y valores.

**Acciones:** marcar como pagada, editar, eliminar con confirmación, activar o desactivar recordatorios, duplicar como nueva obligación.

**Estados vacíos:** historial sin pagos: "Esta obligación aún no tiene pagos registrados. Cuando la marques como pagada, el historial aparecerá aquí". Notas vacías: "Sin notas adicionales para esta obligación".

**Errores:** obligación no encontrada ("Esta obligación ya no existe o fue eliminada") con regreso a la lista; error al guardar cambios con reintento conservando lo editado.

**Loading:** esqueleto del encabezado y del historial durante la carga; confirmación con indicador al marcar como pagada.

**Responsive/mobile:** encabezado fijo con valor y vencimiento siempre visibles, historial en lista vertical y acciones principales en barra inferior al alcance del pulgar.

### 6. Crear/Editar obligación

**Propósito:** crear una obligación nueva o corregir una existente en menos de un minuto, con validación inmediata.

**Información mostrada:** formulario con nombre, valor en COP, categoría (selector con las 9 fijas), fecha de vencimiento, frecuencia, método de pago genérico (tarjeta, transferencia, efectivo, débito automático, otro), notas opcionales e interruptor de recordatorios con offsets {3,1,0}.

**Acciones:** guardar, cancelar sin guardar, previsualizar el próximo vencimiento calculado, activar o desactivar recordatorios.

**Estados vacíos:** en modo creación, las notas muestran el marcador "Agrega una nota opcional, por ejemplo el número de referencia para pagar". En modo edición sin cambios, el botón de guardar permanece deshabilitado con el aviso "Sin cambios por guardar".

**Errores:** validación por campo (nombre obligatorio, valor mayor o igual a cero, fecha obligatoria); mensaje general "Revisa los campos marcados antes de guardar"; error de guardado con reintento sin perder lo digitado.

**Loading:** botón de guardar con indicador y formulario bloqueado durante el envío; cálculo del próximo vencimiento sin recargar la pantalla.

**Responsive/mobile:** formulario en pasos simples o secciones plegables, selector de fecha nativo del sistema, teclado numérico para el valor y botón de guardar fijo al final.

### 7. Calendario

**Propósito:** visualizar el mes con cada obligación ubicada en su fecha de vencimiento y navegar entre meses.

**Información mostrada:** cuadrícula mensual con marcadores por día (cantidad y valor total), distinción entre vencido, próximo y pagado, y panel del día seleccionado con la lista de obligaciones de esa fecha.

**Acciones:** cambiar de mes, seleccionar un día para ver su detalle, abrir una obligación desde el día, ir al mes actual, crear obligación en una fecha tocando un día vacío.

**Estados vacíos:** mes sin obligaciones, con el estado vacío: "Este mes no tiene vencimientos registrados. Toca un día para agregar una obligación en esa fecha". Día vacío seleccionado: "Nada vence este día. Tócalo para programar algo aquí si quieres".

**Errores:** error al cargar el mes con reintento por mes ("No pudimos cargar este mes. Inténtalo de nuevo"); los meses ya cargados se conservan en caché visible.

**Loading:** cuadrícula con marcadores difuminados mientras carga el mes y panel del día con esqueleto breve.

**Responsive/mobile:** en móvil inicia en vista de agenda (lista por días) con opción de cambiar a cuadrícula; días con área táctil amplia y desplazamiento vertical del mes.

### 8. Resumen mensual

**Propósito:** mostrar el total del mes en COP, el desglose por categoría y el estado de pagos para entender a dónde se va el dinero.

**Información mostrada:** total a pagar, total pagado y pendiente, conteo de obligaciones, desglose por las 9 categorías con valores y porcentajes, comparador simple con el mes anterior y lista de las obligaciones del mes.

**Acciones:** cambiar de mes, filtrar por categoría, abrir el detalle de una obligación, exportar el mes en CSV, ir al calendario.

**Estados vacíos:** mes sin datos: "No hay movimientos este mes. Registra tus obligaciones y aquí verás el resumen con totales por categoría". Categoría sin uso: "Sin gastos en esta categoría durante este mes".

**Errores:** error de cálculo o carga con reintento; totales parciales etiquetados como aproximados hasta completar la carga.

**Loading:** tarjetas de totales con esqueleto y desglose por categoría con barras de progreso durante el cálculo.

**Responsive/mobile:** totales primero en tarjetas apiladas, desglose en barras horizontales legibles en pantalla pequeña y lista del mes en formato compacto.

### 9. Notificaciones

**Propósito:** centralizar el historial de recordatorios enviados por correo y su estado, alimentado por `notification_log`.

**Información mostrada:** lista de avisos con obligación relacionada, tipo de aviso (3 días, 1 día o el mismo día), fecha de envío, estado (enviado, pendiente, fallido) y preferencia global de recordatorios.

**Acciones:** reenviar información del aviso, abrir la obligación relacionada, activar o desactivar recordatorios globalmente y por obligación, filtrar por estado.

**Estados vacíos:** sin avisos todavía: "Todavía no hemos enviado recordatorios. Cuando se acerque un vencimiento con avisos activos, los verás listados aquí". Filtro sin resultados: "Ningún aviso coincide con ese estado. Prueba con otro filtro".

**Errores:** error al cargar el historial con reintentar; aviso fallido explicado sin tecnicismos ("Este aviso no pudo enviarse. Revisa que tu correo siga válido en Configuración").

**Loading:** lista con esqueleto de filas y actualización en línea del interruptor global.

**Responsive/mobile:** lista de una columna con estado visible por color y texto, interruptores grandes y filtros en chips horizontales desplazables.

### 10. Configuración

**Propósito:** ajustar preferencias de cuenta, recordatorios y datos personales, incluyendo la exportación y la eliminación de cuenta.

**Información mostrada:** datos de cuenta (correo), zona horaria (`America/Bogota` por defecto), preferencias de recordatorios (offsets {3,1,0} y hora de envío), opciones de exportación CSV, opción de eliminación de cuenta y versión de la aplicación.

**Acciones:** actualizar correo o preferencias, cambiar zona horaria, exportar datos en CSV, eliminar cuenta con doble confirmación, cerrar sesión.

**Estados vacíos:** sin exportaciones previas: "Aún no has exportado tus datos. Genera tu primer archivo CSV cuando lo necesites". Preferencias sin cambios: "Tus ajustes están al día. No hay cambios por guardar".

**Errores:** error al guardar ajustes con reintento conservando valores; eliminación de cuenta con confirmación en dos pasos y mensaje claro de irreversibilidad; error de exportación con reintento.

**Loading:** guardado con indicador por sección sin bloquear el resto de la pantalla; exportación con progreso hasta la descarga.

**Responsive/mobile:** secciones en lista vertical con interruptores nativos grandes, acciones destructivas separadas al final y confirmaciones a pantalla completa.

### 11. Perfil

**Propósito:** mostrar la información básica del usuario del MVP y el estado de su cuenta, sin funciones sociales ni planes de pago en esta fase.

**Información mostrada:** nombre o alias, correo, fecha de registro en formato DD/MM/AAAA, conteo de obligaciones activas y recordatorios activos. Nota visible: la gestión de suscripción del producto (plan Free o Premium) no existe en el MVP y no se muestra aquí.

**Acciones:** editar nombre o alias, cambiar contraseña, ir a configuración, cerrar sesión.

**Estados vacíos:** nombre sin definir: "Aún no tienes un nombre en tu perfil. Agrega cómo quieres que te llamemos". Esta pantalla nunca promete planes ni pagos: si el usuario busca opciones de suscripción del producto, se indica que estarán disponibles en V1.

**Errores:** error al actualizar el perfil con reintento; error al cambiar contraseña con reglas claras (longitud mínima y confirmación coincidente).

**Loading:** campos con esqueleto breve y guardado con indicador en el botón.

**Responsive/mobile:** tarjeta de perfil compacta arriba, acciones en lista vertical y edición en hoja inferior en móvil.

### 12. Suscripciones [V1]

**Propósito:** módulo dedicado de V1 para gestionar suscripciones (plataformas, servicios y membresías) con detección de cambios de precio. Nota de fase: en el MVP las suscripciones se registran como obligaciones recurrentes con categoría Suscripciones; este módulo específico llega en V1 y lo descrito aquí es diseño conceptual breve.

**Diseño conceptual:** lista de suscripciones con costo mensual en COP, fecha de renovación y estado; alertas cuando el precio cambia entre periodos; acciones de pausar, cancelar o convertir en obligación normal. No incluye pagos automáticos: solo registro y recordatorios.

### 13. Cuotas [V1]

**Propósito:** módulo dedicado de V1 para compras a cuotas con seguimiento de cuántas faltan y cuánto resta. Nota de fase: en el MVP una compra a cuotas se registra como obligación con categoría Financiero y el detalle de cuotas en notas; este módulo específico llega en V1 y lo descrito aquí es diseño conceptual breve.

**Diseño conceptual:** plan de cuotas con número total, cuota actual, valor por cuota en COP y saldo restante; barra de progreso por compra; aviso antes de cada cuota con offsets {3,1,0}. No calcula intereses ni otorga crédito: solo organiza el plan que el usuario declara.

### 14. Documentos [V1]

**Propósito:** pantalla de V1 para adjuntar y consultar facturas y soportes con extracción de datos (OCR) y confirmación humana obligatoria. Nota de fase: el MVP no tiene subida de archivos ni OCR; esta pantalla llega en V1 y lo descrito aquí es diseño conceptual breve.

**Diseño conceptual:** visor de documentos por obligación, datos extraídos propuestos (valor, fecha, proveedor) siempre pendientes de confirmación del usuario, y estados de procesamiento (subiendo, procesando, listo para revisar, error de lectura). Incluye validación de tipo de archivo y límites de tamaño por definir en V1.

### 15. Análisis [V2]

**Propósito:** pantalla de V2 para análisis inteligente del comportamiento de pagos con estadística simple y resúmenes opcionales. Nota de fase: fuera del MVP y de V1; lo descrito aquí es diseño conceptual breve sujeto a validación de las fases anteriores.

**Diseño conceptual:** tendencias de gasto por categoría, comparativas entre meses y avisos de patrones (por ejemplo, una categoría que crece tres meses seguidos). No ofrece asesoría financiera: presenta datos descriptivos y deja toda decisión al usuario.

## 4. Textos de estado vacío (referencia rápida)

Los textos concretos usados en las pantallas, listos para copiar a la interfaz. Cada texto de estado vacío está redactado en español cercano y termina con una acción:

1. Onboarding: "Todavía no tienes obligaciones registradas. Crea la primera ahora y activa tus recordatorios para no olvidarla".
2. Dashboard: "No tienes obligaciones por aquí. Registra tu primera factura, arriendo o suscripción para ver tu resumen del mes".
3. Obligaciones sin resultados de filtro: "Ninguna obligación coincide con ese filtro. Ajusta la búsqueda o limpia los filtros para ver todo de nuevo".
4. Calendario de mes vacío: "Este mes no tiene vencimientos registrados. Toca un día para agregar una obligación en esa fecha".
5. Notificaciones sin avisos: "Todavía no hemos enviado recordatorios. Cuando se acerque un vencimiento con avisos activos, los verás listados aquí".

## 5. Convenciones transversales

**Errores.** Formato en dos partes: qué pasó y qué hacer. Ejemplos: "No pudimos guardar los cambios. Revisa tu conexión e inténtalo de nuevo" y "Esta obligación ya no existe o fue eliminada. Vuelve a la lista para continuar". Los errores de validación aparecen junto al campo y los errores de red ofrecen reintentar sin perder lo digitado.

**Loading.** Toda carga de lista usa esqueletos; todo envío usa botón con indicador deshabilitado; ninguna espera bloquea la navegación hacia atrás. El dashboard y el calendario conservan los últimos datos visibles mientras actualizan.

**Responsive y accesibilidad.** Diseño móvil primero con objetivos táctiles amplios, contraste AA con texto mínimo de 16 px en móvil, focos visibles con teclado, etiquetas en cada campo de formulario y mensajes de error anunciados junto al campo correspondiente. La navegación principal (Dashboard, Obligaciones, Calendario, Resumen, Configuración) permanece alcanzable en una barra inferior en móvil y lateral en escritorio.

**Lo que no existe en el MVP.** No hay pantallas de pago ni de movimiento de dinero, no hay conexión con bancos, no hay OCR ni importación de facturas, no hay análisis inteligente y no hay gestión de planes Free o Premium. Las pantallas 12 a 15 describen V1 y V2 únicamente como diseño conceptual y no deben interpretarse como compromiso de interfaz final.
