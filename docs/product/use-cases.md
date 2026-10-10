# Casos de uso principales de Shadow-Net Bills

Shadow-Net Bills ayuda a personas en Colombia a registrar sus obligaciones financieras, recordar sus vencimientos por correo electrónico y revisar su calendario y resumen mensual. Este documento describe los 16 casos de uso principales del producto: 12 del MVP y 4 de fases posteriores marcados con su fase. El sistema nunca mueve dinero ni se conecta a bancos: solo organiza recordatorios y el usuario paga por sus propios medios.

## Convenciones

Cada caso de uso usa el mismo formato para facilitar su lectura y trazabilidad con los requisitos.

- Identificador: `UC-XX` con numeración consecutiva de dos dígitos.
- Encabezado: `### UC-XX — <título>` con raya larga.
- Campos obligatorios por caso: Actor, Precondiciones, Flujo principal (pasos numerados), Flujos alternativos y Excepciones, Postcondiciones.
- Fase: cada caso indica entre paréntesis si pertenece al MVP, a V1 o a V2. Los casos UC-15 y UC-16 son de fases futuras y no forman parte del MVP.
- Terminología canónica: la tabla se llama `obligations` con columna `next_due_date`; la API base es `/api/v1/obligations`; las categorías fijas son Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno y Otros; los recordatorios usan los desplazamientos {3,1,0} días; los métodos de pago genéricos son tarjeta, transferencia, efectivo, débito automático u otro.
- Formato de moneda: pesos colombianos como `$20.000`. Fechas en formato `DD/MM/AAAA`.

## Actores

- Usuario registrado: persona con cuenta activa y sesión iniciada que gestiona sus propias obligaciones, calendario, resumen y configuración.
- Visitante: persona sin cuenta o sin sesión iniciada que solo puede registrarse, iniciar sesión o recuperar su contraseña. No accede a obligaciones ajenas ni a datos de otros usuarios.
- Sistema/scheduler: proceso automático que ejecuta los recordatorios programados con desplazamientos {3,1,0} días y genera los siguientes vencimientos de obligaciones recurrentes. Actúa sin intervención del usuario y deja registro en `notification_log`.

## Casos de uso del MVP

### UC-01 — Registrar obligación (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Conoce las 9 categorías fijas disponibles.
- Flujo principal:
  1. El usuario abre la lista de obligaciones y elige la opción de crear una nueva.
  2. El sistema muestra el formulario con nombre, monto en COP, categoría, fecha de vencimiento, frecuencia (única, semanal, mensual, trimestral o anual), método de pago genérico y notas.
  3. El usuario completa los campos obligatorios: nombre, monto, categoría y fecha de vencimiento.
  4. El usuario confirma el registro.
  5. El sistema valida los datos y guarda la obligación en la tabla `obligations` con estado pendiente.
  6. El sistema muestra la obligación creada en la lista y confirma la operación.
- Flujos alternativos y Excepciones:
  - Datos incompletos o monto negativo: el sistema rechaza el guardado, indica el campo con error y no crea el registro. Excepción E1.
  - Fecha de vencimiento pasada: el sistema advierte que la fecha ya pasó y pide confirmación antes de guardar. Alternativo A1.
  - Error de conexión: el sistema muestra un mensaje de reintento y conserva los datos del formulario. Excepción E2.
- Postcondiciones: la obligación queda registrada como pendiente con su vencimiento y aparece en la lista y en el calendario del mes.

### UC-02 — Editar obligación (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Existe al menos una obligación propia.
- Flujo principal:
  1. El usuario abre el detalle de una obligación y elige editarla.
  2. El sistema muestra el formulario con los valores actuales.
  3. El usuario modifica nombre, monto, categoría, fecha, frecuencia, método de pago genérico o notas.
  4. El usuario guarda los cambios.
  5. El sistema valida y actualiza el registro en `obligations`.
  6. El sistema confirma la actualización y muestra los nuevos valores.
- Flujos alternativos y Excepciones:
  - El usuario cancela sin guardar: el sistema descarta los cambios y conserva los valores anteriores. Alternativo A1.
  - Datos inválidos (monto negativo o fecha con formato incorrecto): el sistema indica el error y no guarda. Excepción E1.
  - La obligación ya no existe (eliminada en otra sesión): el sistema informa que el registro no se encontró. Excepción E2.
- Postcondiciones: la obligación queda actualizada y los recordatorios pendientes usan los nuevos datos.

### UC-03 — Eliminar obligación con confirmación (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Existe al menos una obligación propia.
- Flujo principal:
  1. El usuario elige eliminar una obligación desde la lista o el detalle.
  2. El sistema muestra un diálogo de confirmación con el nombre y el monto de la obligación.
  3. El usuario confirma la eliminación.
  4. El sistema elimina el registro y sus recordatorios pendientes asociados.
  5. El sistema confirma la eliminación y actualiza la lista.
- Flujos alternativos y Excepciones:
  - El usuario cancela la confirmación: el sistema no elimina nada. Alternativo A1.
  - Error de conexión durante el borrado: el sistema informa el fallo y conserva el registro. Excepción E1.
- Postcondiciones: la obligación ya no aparece en lista, calendario ni resumen. No se generan más recordatorios para ella.

### UC-04 — Marcar obligación como pagada (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Existe una obligación propia en estado pendiente.
- Flujo principal:
  1. El usuario abre la obligación pendiente y elige la opción de marcar como pagada.
  2. El sistema pide la fecha de pago y el método de pago genérico usado (tarjeta, transferencia, efectivo, débito automático u otro).
  3. El usuario confirma el pago con esos datos.
  4. El sistema registra el pago en el historial y cambia el estado de la obligación a pagada.
  5. El sistema cancela los recordatorios pendientes de ese vencimiento y muestra la confirmación.
- Flujos alternativos y Excepciones:
  - El usuario ya había marcado ese vencimiento como pagado: el sistema advierte del posible duplicado y pide confirmación. Alternativo A1.
  - Fecha de pago futura: el sistema advierte y pide confirmación explícita. Alternativo A2.
  - Error al guardar el historial: el sistema informa el fallo y mantiene el estado pendiente. Excepción E1.
- Postcondiciones: el vencimiento queda en estado pagado, el historial incluye el nuevo registro y el resumen mensual refleja el pago.

### UC-05 — Generar siguiente vencimiento de obligación recurrente (MVP)

- Actor: sistema/scheduler con participación del usuario registrado.
- Precondiciones: existe una obligación recurrente (semanal, mensual, trimestral o anual) marcada como pagada. La tabla `obligations` incluye la columna `next_due_date`.
- Flujo principal:
  1. Al marcar como pagada una obligación recurrente, el sistema calcula el siguiente vencimiento según su frecuencia.
  2. El sistema crea el nuevo vencimiento pendiente con el mismo nombre, monto, categoría y método de pago genérico.
  3. El sistema guarda el valor calculado en `next_due_date` y programa sus recordatorios {3,1,0}.
  4. El sistema muestra al usuario el nuevo vencimiento generado.
- Flujos alternativos y Excepciones:
  - Obligación de frecuencia única: el sistema no genera ningún vencimiento adicional. Alternativo A1.
  - El usuario edita el monto antes del siguiente vencimiento: el nuevo vencimiento conserva el último monto guardado. Alternativo A2.
  - Fallo al programar recordatorios: el vencimiento se crea igual y el sistema reintenta la programación. Excepción E1.
- Postcondiciones: existe un nuevo vencimiento pendiente con sus recordatorios programados y la cadena de recurrencia queda continua.

### UC-06 — Configurar recordatorios (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Existe al menos una obligación propia.
- Flujo principal:
  1. El usuario abre la configuración de recordatorios de una obligación o la configuración general.
  2. El sistema muestra las opciones: activar o desactivar recordatorios y los avisos a 3, 1 y 0 días del vencimiento.
  3. El usuario activa o desactiva cada aviso y guarda la configuración.
  4. El sistema valida y guarda la preferencia.
  5. El sistema confirma que los próximos avisos seguirán la nueva configuración.
- Flujos alternativos y Excepciones:
  - El usuario desactiva todos los avisos: el sistema advierte que no recibirá recordatorios de esa obligación y pide confirmación. Alternativo A1.
  - Correo no verificado: el sistema pide verificar el correo antes de activar avisos por email. Excepción E1.
- Postcondiciones: la configuración queda guardada y el scheduler usa los desplazamientos {3,1,0} según lo elegido.

### UC-07 — Recibir notificación de vencimiento por correo electrónico (MVP)

- Actor: sistema/scheduler (el usuario registrado es el destinatario).
- Precondiciones: existe una obligación pendiente con recordatorios activos y correo verificado. El scheduler se ejecuta todos los días.
- Flujo principal:
  1. El scheduler revisa los vencimientos próximos según los desplazamientos {3,1,0} días.
  2. El scheduler compone el correo con nombre, monto, categoría y fecha de vencimiento de la obligación.
  3. El scheduler envía el correo y registra el envío en `notification_log` con clave de idempotencia.
  4. El usuario recibe el correo antes del vencimiento.
- Flujos alternativos y Excepciones:
  - La obligación se marcó como pagada antes del envío: el scheduler omite el aviso pendiente. Alternativo A1.
  - El correo rebota o el servicio de envío falla: el scheduler registra el fallo y reintenta según su política. Excepción E1.
  - Envío duplicado por reintento: la clave de idempotencia evita el segundo envío. Excepción E2.
- Postcondiciones: el aviso queda registrado en `notification_log` y el usuario dispone de la información para pagar por sus propios medios.

### UC-08 — Consultar calendario mensual (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Puede no tener obligaciones registradas.
- Flujo principal:
  1. El usuario abre la vista de calendario.
  2. El sistema muestra el mes actual con cada obligación ubicada en su fecha de vencimiento y su estado (pendiente o pagada).
  3. El usuario navega entre meses anteriores y siguientes.
  4. El usuario selecciona un día para ver el detalle de sus vencimientos.
- Flujos alternativos y Excepciones:
  - Mes sin obligaciones: el sistema muestra un estado vacío con una invitación a registrar la primera obligación. Alternativo A1.
  - Error al cargar los datos: el sistema muestra un mensaje de reintento sin perder el mes seleccionado. Excepción E1.
- Postcondiciones: el usuario visualiza sus vencimientos del mes sin modificar ningún dato.

### UC-09 — Ver resumen mensual (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada.
- Flujo principal:
  1. El usuario abre el resumen del mes.
  2. El sistema calcula el total pendiente, el total pagado y el desglose por categoría (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno y Otros).
  3. El sistema presenta las tarjetas del panel y la lista de vencimientos del mes.
  4. El usuario cambia de mes para comparar periodos.
- Flujos alternativos y Excepciones:
  - Mes sin movimientos: el sistema muestra totales en cero con un mensaje explicativo. Alternativo A1.
  - Datos en caché desactualizados tras un pago reciente: el sistema recalcula al abrir la vista. Alternativo A2.
- Postcondiciones: el usuario conoce su situación del mes. Ningún dato se modifica por consultar el resumen.

### UC-10 — Exportar datos en CSV (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Existe al menos una obligación o un pago registrado.
- Flujo principal:
  1. El usuario elige la opción de exportar sus datos.
  2. El sistema ofrece el rango (mes actual u otro periodo) y el formato CSV.
  3. El usuario confirma la exportación.
  4. El sistema genera el archivo CSV con nombre, monto, categoría, vencimiento, estado y método de pago genérico.
  5. El sistema entrega el archivo para descarga.
- Flujos alternativos y Excepciones:
  - Periodo sin datos: el sistema informa que no hay registros para exportar y no genera archivo vacío. Alternativo A1.
  - Fallo en la generación del archivo: el sistema muestra un error y permite reintentar. Excepción E1.
- Postcondiciones: el usuario obtiene una copia CSV de sus propios datos para respaldo o análisis externo.

### UC-11 — Eliminar cuenta (MVP)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada y conoce las consecuencias del borrado.
- Flujo principal:
  1. El usuario abre la configuración de cuenta y elige eliminar su cuenta.
  2. El sistema explica que se borrarán obligaciones, historial, configuración y recordatorios, y pide confirmación escrita.
  3. El usuario confirma la eliminación.
  4. El sistema elimina los datos del usuario y cierra todas sus sesiones.
  5. El sistema muestra la confirmación final del borrado.
- Flujos alternativos y Excepciones:
  - El usuario cancela la confirmación: la cuenta y los datos quedan intactos. Alternativo A1.
  - Fallo parcial del borrado: el sistema informa el error y mantiene un registro interno para completar la eliminación. Excepción E1.
- Postcondiciones: la cuenta queda eliminada y el usuario ya no puede iniciar sesión con esas credenciales.

### UC-12 — Recuperar contraseña (MVP)

- Actor: visitante.
- Precondiciones: existe una cuenta registrada con el correo indicado. El visitante no tiene sesión iniciada.
- Flujo principal:
  1. El visitante abre la opción de recuperar contraseña e ingresa su correo.
  2. El sistema genera un enlace temporal de restablecimiento y lo envía al correo.
  3. El visitante abre el enlace y define una nueva contraseña.
  4. El sistema valida la nueva contraseña y la guarda.
  5. El sistema confirma el cambio e invita a iniciar sesión.
- Flujos alternativos y Excepciones:
  - Correo no registrado: el sistema muestra un mensaje genérico sin revelar si la cuenta existe. Alternativo A1.
  - Enlace vencido o ya usado: el sistema pide solicitar uno nuevo. Excepción E1.
  - Demasiados intentos seguidos: el sistema aplica un bloqueo temporal por seguridad. Excepción E2.
- Postcondiciones: la contraseña queda actualizada y los enlaces anteriores quedan invalidados.

### UC-13 — Registrar suscripción (MVP como obligación recurrente; módulo específico en V1)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Nota de alcance: en el MVP no existe un módulo de suscripciones; toda suscripción se registra como obligación recurrente con categoría Suscripciones y el módulo específico llega en V1.
- Flujo principal:
  1. El usuario crea una obligación con categoría Suscripciones (por ejemplo, plataforma de música de $18.900 al mes).
  2. El usuario define la frecuencia mensual y la fecha del próximo cobro.
  3. El usuario guarda el registro con notas sobre el plan contratado.
  4. El sistema programa los recordatorios {3,1,0} y la generación del siguiente vencimiento.
  5. Cada mes, al marcar como pagada la suscripción, el sistema genera el siguiente vencimiento.
- Flujos alternativos y Excepciones:
  - El usuario cancela la suscripción con el proveedor: marca la obligación como finalizada o la elimina con confirmación. Alternativo A1.
  - Cambio de precio del plan: el usuario edita el monto y los próximos vencimientos usan el nuevo valor. Alternativo A2.
- Postcondiciones: la suscripción queda controlada como obligación recurrente hasta que el usuario la finalice. En V1 migrará al módulo específico sin pérdida de historial.

### UC-14 — Registrar compra a cuotas (MVP como obligación con notas; módulo específico en V1)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Nota de alcance: en el MVP no existe un módulo de cuotas; la compra a cuotas se registra como obligación con categoría Financiero y el detalle de cuotas en notas. El módulo específico llega en V1.
- Flujo principal:
  1. El usuario crea una obligación con categoría Financiero por el valor de la cuota mensual (por ejemplo, $200.000 de 12 cuotas).
  2. El usuario indica en las notas el número total de cuotas y la cuota actual (por ejemplo, cuota 3 de 12).
  3. El usuario define frecuencia mensual y guarda el registro.
  4. Cada mes, al marcar la cuota como pagada, el sistema genera el siguiente vencimiento y el usuario actualiza el número de cuota en notas.
- Flujos alternativos y Excepciones:
  - Pago anticipado de varias cuotas: el usuario edita las notas y la fecha del siguiente vencimiento. Alternativo A1.
  - Error en el número de cuota registrado: el usuario corrige las notas sin afectar el historial de pagos. Alternativo A2.
- Postcondiciones: la compra a cuotas queda seguida mes a mes como obligación recurrente. En V1 migrará al módulo específico con cálculo automático de saldo.

## Casos de uso de fases futuras

### UC-15 — Importar factura (V1)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada. Fase V1: el MVP no incluye importación; este caso describe el comportamiento previsto cuando la importación esté disponible. El usuario dispone de un archivo de factura electrónica en XML de la DIAN o de una imagen legible.
- Flujo principal:
  1. El usuario elige la opción de importar factura disponible en V1.
  2. El usuario adjunta el archivo XML o la imagen de la factura.
  3. El sistema extrae nombre del emisor, monto, fecha de vencimiento y CUFE cuando aplica.
  4. El sistema muestra los datos extraídos para confirmación humana obligatoria.
  5. El usuario corrige si hace falta y confirma la creación de la obligación.
  6. El sistema crea la obligación con la categoría sugerida y guarda el documento adjunto.
- Flujos alternativos y Excepciones:
  - Archivo ilegible o con formato no soportado: el sistema informa el problema y ofrece el registro manual como alternativa. Alternativo A1.
  - Extracción con confianza baja: el sistema marca los campos dudosos y exige revisión antes de guardar. Excepción E1.
  - Documento duplicado (mismo CUFE ya importado): el sistema advierte del duplicado y pide confirmación. Excepción E2.
- Postcondiciones: la obligación queda creada a partir de la factura con su documento de respaldo asociado.

### UC-16 — Revisar análisis de gastos (V2)

- Actor: usuario registrado.
- Precondiciones: el usuario tiene sesión iniciada y cuenta con al menos tres meses de historial de pagos. Fase V2: el MVP y V1 no incluyen análisis inteligente; este caso describe la vista prevista de V2.
- Flujo principal:
  1. El usuario abre la vista de análisis disponible en V2.
  2. El sistema presenta tendencias de gasto por categoría, comparativa entre meses y avisos de cambios de precio detectados por comparación determinista.
  3. El usuario filtra por categoría y periodo.
  4. El usuario revisa el detalle de cada tendencia sin recibir recomendaciones financieras automáticas.
- Flujos alternativos y Excepciones:
  - Historial insuficiente: el sistema indica cuántos meses faltan para activar el análisis. Alternativo A1.
  - Datos atípicos por un pago extraordinario: el sistema señala el valor atípico para que el usuario lo interprete. Alternativo A2.
- Postcondiciones: el usuario comprende la evolución de sus gastos. El sistema no genera asesoría financiera ni modifica obligaciones.
