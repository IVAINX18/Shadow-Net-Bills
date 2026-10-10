# Especificación de API REST de Shadow-Net Bills

Shadow-Net Bills expone una API REST que permite al frontend web gestionar la cuenta del usuario, sus obligaciones financieras, las categorías fijas, el resumen mensual, el calendario de vencimientos, la configuración y la exportación de datos. Esta especificación cubre únicamente el MVP manual-primero: la API organiza recordatorios y registros, nunca mueve dinero, nunca se conecta a bancos y no incluye funciones de IA. Los datos de ejemplo de este documento son ficticios y solo ilustran los esquemas.

## Convenciones generales

Todas las reglas de esta sección aplican a cada endpoint del MVP, salvo que el endpoint indique lo contrario.

- Base: todas las rutas empiezan por `/api/v1`. El host depende del ambiente (desarrollo o producción) y se configura por variables de entorno.
- Formato: las peticiones con cuerpo y todas las respuestas usan `Content-Type: application/json; charset=utf-8`, salvo la exportación CSV que responde `text/csv; charset=utf-8`.
- Autenticación: esquema `Bearer` con el JWT de acceso emitido por Supabase Auth. El frontend obtiene el token en el login y lo envía en cada petición protegida con la cabecera `Authorization: Bearer <access_token>`. La API valida el JWT en cada petición y nunca acepta peticiones sin token en rutas protegidas.
- Errores: toda respuesta de error usa el objeto `{code, message, details}`. El campo `code` es un identificador estable en mayúsculas con guiones bajos (por ejemplo `OBLIGATION_NOT_FOUND`), `message` es un texto legible en español y `details` lleva el detalle por campo o es `null` cuando no hay detalle adicional.
- Paginación por cursor (opcional): los listados aceptan los parámetros `cursor` y `limit`. `limit` vale 20 por defecto y 100 como máximo. La respuesta incluye `next_cursor`, que es `null` cuando no hay más páginas. El cliente envía ese valor como `cursor` para pedir la página siguiente.
- Rate limiting: límite de 100 req/min por usuario autenticado. Al superarlo, la API responde `429` con la cabecera `Retry-After` en segundos. Los endpoints de autenticación tienen un límite adicional más estricto contra fuerza bruta (ver la sección de autenticación).
- Moneda y fechas: los montos se expresan en pesos colombianos como número con dos decimales (por ejemplo `95000.00`, que se presenta como $95.000). Las fechas usan el formato `DD/MM/AAAA` (por ejemplo `15/10/2026`) y el mes se pide como `MM-AAAA` (por ejemplo `10-2026`). La zona horaria de referencia es `America/Bogota`.
- Versionado: los cambios incompatibles se publican como una versión nueva (`/api/v2`) y la versión anterior se mantiene durante un periodo de transición que se comunica con antelación.
- Nombres canónicos: la tabla principal se llama `obligations` con columna `next_due_date`; las 9 categorías fijas son Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno y Otros; los recordatorios usan los desplazamientos {3,1,0} días; los métodos de pago genéricos son tarjeta, transferencia, efectivo, débito automático y otro. Estos valores aparecen tal cual en los esquemas.

## Autenticación

La autenticación del MVP usa correo y contraseña gestionados por Supabase Auth. La API expone cuatro operaciones: registro (signup), inicio de sesión (login), cierre de sesión (logout) y restablecimiento de contraseña (reset). Tras un login exitoso, la API devuelve `access_token`, `refresh_token` y `expires_in` en segundos. El `access_token` viaja como `Bearer` y el `refresh_token` solo se usa contra el endpoint de renovación. Por seguridad, los endpoints de registro, login y reset aplican un límite adicional de 10 intentos por minuto por dirección IP y responden `429` al superarlo. Las contraseñas nunca se devuelven en ninguna respuesta y los tokens solo se entregan al dueño de la cuenta.

### Autenticación

Registro de una cuenta nueva.

```http
POST /api/v1/auth/signup
Content-Type: application/json
```

```json
{
  "email": "ana.ejemplo@correo.co",
  "password": "ClaveSegura123*"
}
```

Respuesta `201`:

```json
{
  "user": {
    "id": "a1b2c3d4-0000-4000-8000-aaaaaaaa0001",
    "email": "ana.ejemplo@correo.co",
    "created_at": "07/10/2026"
  },
  "message": "Cuenta creada. Revisa tu correo para confirmarla."
}
```

Inicio de sesión con correo y contraseña.

```http
POST /api/v1/auth/login
Content-Type: application/json
```

```json
{
  "email": "ana.ejemplo@correo.co",
  "password": "ClaveSegura123*"
}
```

Respuesta `200`:

```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.ejemplo",
  "refresh_token": "r1-ejemplo-de-token-de-renovacion",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Cierre de sesión. Requiere `Bearer` y revoca el token actual.

```http
POST /api/v1/auth/logout
Authorization: Bearer <access_token>
```

Respuesta `204` sin cuerpo.

Solicitud de restablecimiento de contraseña. Siempre responde `202` para no revelar si el correo existe.

```http
POST /api/v1/auth/reset
Content-Type: application/json
```

```json
{
  "email": "ana.ejemplo@correo.co"
}
```

Respuesta `202`:

```json
{
  "message": "Si el correo existe, recibirás un enlace de restablecimiento."
}
```

## Endpoints de obligaciones

Las obligaciones son el recurso central y viven en la tabla `obligations`. Cada obligación pertenece a un solo usuario y la API solo devuelve las del dueño del token. La lista acepta filtros por `status` (pending, paid, overdue) y por `category` (una de las 9 fijas). Marcar como pagada es una operación dedicada que registra el pago en el historial y, si la obligación es recurrente, genera el siguiente vencimiento en `next_due_date`.

### Obligaciones: listar y crear

Listar las obligaciones propias con filtros opcionales y paginación por cursor.

```http
GET /api/v1/obligations?status=pending&category=Servicios%20p%C3%BAblicos&limit=20
Authorization: Bearer <access_token>
```

Respuesta `200`:

```json
{
  "data": [
    {
      "id": "0b11c101-0000-4000-8000-bbbbbbbb0001",
      "name": "Energía Enel",
      "amount": 95400.00,
      "currency": "COP",
      "category": "Servicios públicos",
      "due_date": "15/10/2026",
      "next_due_date": "15/11/2026",
      "frequency": "monthly",
      "payment_method": "débito automático",
      "notes": "Se descuenta de la cuenta principal",
      "reminder_enabled": true,
      "reminder_days": [3, 1, 0],
      "status": "pending",
      "created_at": "01/10/2026",
      "updated_at": "01/10/2026"
    },
    {
      "id": "0b11c101-0000-4000-8000-bbbbbbbb0002",
      "name": "Arriendo apartamento",
      "amount": 850000.00,
      "currency": "COP",
      "category": "Vivienda",
      "due_date": "05/10/2026",
      "next_due_date": "05/11/2026",
      "frequency": "monthly",
      "payment_method": "transferencia",
      "notes": null,
      "reminder_enabled": true,
      "reminder_days": [3, 1, 0],
      "status": "pending",
      "created_at": "01/10/2026",
      "updated_at": "01/10/2026"
    }
  ],
  "next_cursor": "Y3Vyc29yLWVqZW1wbG8tMw",
  "limit": 20
}
```

Crear una obligación nueva. Los campos `name`, `amount`, `category` y `due_date` son obligatorios.

```http
POST /api/v1/obligations
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "name": "Plan celular Claro",
  "amount": 65900.00,
  "currency": "COP",
  "category": "Telecomunicaciones",
  "due_date": "20/10/2026",
  "frequency": "monthly",
  "payment_method": "tarjeta",
  "notes": "Plan de 60 GB",
  "reminder_enabled": true,
  "reminder_days": [3, 1, 0]
}
```

Respuesta `201` con la obligación creada (mismo esquema del listado, con `status` en `pending` y su `id` generado).

Valores aceptados: `frequency` es `one_time`, `weekly`, `monthly`, `quarterly` o `yearly`; `payment_method` es `tarjeta`, `transferencia`, `efectivo`, `débito automático` u `otro`; `category` es una de las 9 fijas; `amount` debe ser mayor o igual a cero.

### Obligaciones: detalle, actualización y eliminación

Obtener una obligación por su identificador. Solo el dueño puede verla; otro usuario recibe `404` para no revelar existencias ajenas.

```http
GET /api/v1/obligations/0b11c101-0000-4000-8000-bbbbbbbb0001
Authorization: Bearer <access_token>
```

Respuesta `200` con el objeto de obligación del ejemplo anterior.

Actualizar una obligación existente con el esquema completo de campos editables.

```http
PUT /api/v1/obligations/0b11c101-0000-4000-8000-bbbbbbbb0001
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "name": "Energía Enel",
  "amount": 102300.00,
  "category": "Servicios públicos",
  "due_date": "15/10/2026",
  "frequency": "monthly",
  "payment_method": "débito automático",
  "notes": "Subió respecto al mes pasado",
  "reminder_enabled": true,
  "reminder_days": [3, 1, 0]
}
```

Respuesta `200` con el objeto actualizado y `updated_at` con la fecha de hoy.

Eliminar una obligación con confirmación previa en el cliente. La operación es definitiva.

```http
DELETE /api/v1/obligations/0b11c101-0000-4000-8000-bbbbbbbb0002
Authorization: Bearer <access_token>
```

Respuesta `204` sin cuerpo. Si el identificador no existe o pertenece a otro usuario, responde `404` con el objeto de error.

### Obligaciones: marcar como pagada

Registra el pago de una obligación pendiente o vencida. Guarda el monto, la fecha y el método en el historial de pagos y, cuando la frecuencia es recurrente, calcula el siguiente vencimiento en `next_due_date`. Esta operación no mueve dinero: solo deja constancia de que el usuario ya pagó por sus propios medios.

```http
POST /api/v1/obligations/0b11c101-0000-4000-8000-bbbbbbbb0001/pay
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "paid_at": "07/10/2026",
  "amount": 95400.00,
  "payment_method": "débito automático",
  "notes": "Pago confirmado en la app del banco"
}
```

Respuesta `200`:

```json
{
  "obligation": {
    "id": "0b11c101-0000-4000-8000-bbbbbbbb0001",
    "name": "Energía Enel",
    "status": "paid",
    "next_due_date": "15/11/2026"
  },
  "payment": {
    "id": "c2d3e401-0000-4000-8000-cccccccc0001",
    "obligation_id": "0b11c101-0000-4000-8000-bbbbbbbb0001",
    "amount": 95400.00,
    "currency": "COP",
    "paid_at": "07/10/2026",
    "payment_method": "débito automático"
  }
}
```

Si la obligación ya está pagada, responde `409` con código `ALREADY_PAID`.

### Categorías

Devuelve las 9 categorías fijas del sistema con el conteo de obligaciones activas del usuario en cada una. No admite creación ni edición en el MVP.

```http
GET /api/v1/categories
Authorization: Bearer <access_token>
```

Respuesta `200`:

```json
{
  "data": [
    {"name": "Vivienda", "obligations_count": 2},
    {"name": "Servicios públicos", "obligations_count": 3},
    {"name": "Telecomunicaciones", "obligations_count": 1},
    {"name": "Suscripciones", "obligations_count": 4},
    {"name": "Financiero", "obligations_count": 1},
    {"name": "Seguros", "obligations_count": 1},
    {"name": "Educación", "obligations_count": 0},
    {"name": "Gobierno", "obligations_count": 1},
    {"name": "Otros", "obligations_count": 0}
  ]
}
```

### Tablero y resumen mensual

Resumen agregado del mes para las tarjetas del tablero: total por vencer, total pagado, total vencido y desagregación por categoría. El parámetro `month` usa el formato `MM-AAAA` y por defecto es el mes actual.

```http
GET /api/v1/summary?month=10-2026
Authorization: Bearer <access_token>
```

Respuesta `200`:

```json
{
  "month": "10-2026",
  "totals": {
    "pending": 1015300.00,
    "paid": 420000.00,
    "overdue": 65900.00,
    "currency": "COP"
  },
  "by_category": [
    {"category": "Vivienda", "pending": 850000.00, "paid": 0.00},
    {"category": "Servicios públicos", "pending": 95400.00, "paid": 120000.00},
    {"category": "Telecomunicaciones", "pending": 69900.00, "paid": 0.00}
  ],
  "upcoming": [
    {
      "id": "0b11c101-0000-4000-8000-bbbbbbbb0001",
      "name": "Energía Enel",
      "amount": 95400.00,
      "due_date": "15/10/2026",
      "status": "pending"
    }
  ]
}
```

### Calendario

Vencimientos del mes agrupados por día para la vista de calendario mensual. Usa el mismo formato `MM-AAAA` que el resumen.

```http
GET /api/v1/calendar?month=10-2026
Authorization: Bearer <access_token>
```

Respuesta `200`:

```json
{
  "month": "10-2026",
  "days": [
    {
      "date": "05/10/2026",
      "obligations": [
        {
          "id": "0b11c101-0000-4000-8000-bbbbbbbb0002",
          "name": "Arriendo apartamento",
          "amount": 850000.00,
          "status": "pending"
        }
      ]
    },
    {
      "date": "15/10/2026",
      "obligations": [
        {
          "id": "0b11c101-0000-4000-8000-bbbbbbbb0001",
          "name": "Energía Enel",
          "amount": 95400.00,
          "status": "pending"
        }
      ]
    }
  ]
}
```

### Configuración

Preferencias del usuario: recordatorios, hora de envío y formato de presentación. La lectura y la actualización usan el mismo esquema.

```http
GET /api/v1/settings
Authorization: Bearer <access_token>
```

Respuesta `200`:

```json
{
  "reminder_enabled": true,
  "reminder_days": [3, 1, 0],
  "reminder_hour": "07:00",
  "timezone": "America/Bogota",
  "currency": "COP",
  "language": "es-CO"
}
```

```http
PUT /api/v1/settings
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "reminder_enabled": true,
  "reminder_days": [3, 1, 0],
  "reminder_hour": "08:00",
  "timezone": "America/Bogota",
  "currency": "COP",
  "language": "es-CO"
}
```

Respuesta `200` con el objeto guardado.

### Exportación

Descarga todas las obligaciones y su historial de pagos en CSV con codificación UTF-8. Acepta los mismos filtros `status` y `category` del listado.

```http
GET /api/v1/export/csv?status=paid
Authorization: Bearer <access_token>
Accept: text/csv
```

Respuesta `200` con `Content-Type: text/csv; charset=utf-8`:

```text
id,nombre,monto,moneda,categoria,vencimiento,estado,pagado_el,metodo
0b11c101-0000-4000-8000-bbbbbbbb0001,Energía Enel,95400.00,COP,Servicios públicos,15/10/2026,paid,07/10/2026,débito automático
0b11c101-0000-4000-8000-bbbbbbbb0002,Arriendo apartamento,850000.00,COP,Vivienda,05/10/2026,pending,,,transferencia
```

### Cuenta

Operaciones sobre la propia cuenta: descarga de datos personales y borrado definitivo. El borrado exige confirmación con la contraseña y elimina obligaciones, pagos, configuración y perfil del usuario autenticado. Es irreversible.

Descargar los datos de la cuenta en JSON:

```http
GET /api/v1/account/export
Authorization: Bearer <access_token>
```

Respuesta `200`:

```json
{
  "profile": {
    "email": "ana.ejemplo@correo.co",
    "created_at": "07/10/2026"
  },
  "obligations": [
    {
      "name": "Energía Enel",
      "amount": 95400.00,
      "category": "Servicios públicos",
      "due_date": "15/10/2026",
      "status": "paid"
    }
  ],
  "payments": [
    {
      "obligation_name": "Energía Enel",
      "amount": 95400.00,
      "paid_at": "07/10/2026"
    }
  ],
  "settings": {
    "reminder_days": [3, 1, 0],
    "timezone": "America/Bogota"
  }
}
```

Eliminar la cuenta propia de forma definitiva:

```http
DELETE /api/v1/account
Authorization: Bearer <access_token>
Content-Type: application/json
```

```json
{
  "password": "ClaveSegura123*",
  "confirmation": "ELIMINAR MI CUENTA"
}
```

Respuesta `204` sin cuerpo.

## Esquemas comunes

El objeto de error aparece en cada respuesta fallida con los campos `code`, `message` y `details`.

```json
{
  "code": "OBLIGATION_NOT_FOUND",
  "message": "La obligación no existe o no te pertenece.",
  "details": null
}
```

Ejemplo de error de validación con detalle por campo:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Hay campos con valores inválidos.",
  "details": {
    "amount": "Debe ser mayor o igual a cero.",
    "due_date": "Usa el formato DD/MM/AAAA."
  }
}
```

## Códigos de error

La tabla resume cada estado HTTP, su código interno, cuándo ocurre y un ejemplo de mensaje.

| HTTP | code | Cuándo ocurre | Ejemplo de message |
| --- | --- | --- | --- |
| 400 | BAD_REQUEST | Petición mal formada o parámetro con formato inválido | El parámetro month debe usar el formato MM-AAAA. |
| 401 | UNAUTHORIZED | Falta el token Bearer o el token expiró | Tu sesión expiró, inicia sesión de nuevo. |
| 403 | FORBIDDEN | El token es válido pero no tiene permiso sobre el recurso | No tienes permiso para esta operación. |
| 404 | NOT_FOUND / OBLIGATION_NOT_FOUND | El recurso no existe o pertenece a otro usuario | La obligación no existe o no te pertenece. |
| 409 | ALREADY_PAID / EMAIL_TAKEN | Conflicto de estado: obligación ya pagada o correo ya registrado | Esta obligación ya está marcada como pagada. |
| 422 | VALIDATION_ERROR | Los datos no pasan la validación de negocio | Hay campos con valores inválidos. |
| 429 | RATE_LIMITED | Se superó el límite de 100 req/min por usuario (o 10/min en autenticación) | Superaste el límite de peticiones, espera unos segundos. |
| 500 | INTERNAL_ERROR | Fallo inesperado del servidor | Ocurrió un error interno, intenta más tarde. |

Notas: el `429` incluye la cabecera `Retry-After` con los segundos de espera. El `500` nunca expone trazas ni secretos, solo el objeto de error genérico.

## Ejemplos curl de extremo a extremo

Registro e inicio de sesión para obtener el token Bearer.

```bash
curl -X POST https://api.ejemplo.co/api/v1/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email":"ana.ejemplo@correo.co","password":"ClaveSegura123*"}'

curl -X POST https://api.ejemplo.co/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"ana.ejemplo@correo.co","password":"ClaveSegura123*"}'
```

Crear una obligación y listar las pendientes de una categoría.

```bash
TOKEN="pega-aqui-tu-access-token"

curl -X POST https://api.ejemplo.co/api/v1/obligations \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Internet hogar","amount":89900.00,"currency":"COP","category":"Telecomunicaciones","due_date":"25/10/2026","frequency":"monthly","payment_method":"tarjeta","reminder_enabled":true,"reminder_days":[3,1,0]}'

curl "https://api.ejemplo.co/api/v1/obligations?status=pending&category=Telecomunicaciones&limit=20" \
  -H "Authorization: Bearer $TOKEN"
```

Marcar como pagada y consultar el resumen del mes.

```bash
curl -X POST https://api.ejemplo.co/api/v1/obligations/0b11c101-0000-4000-8000-bbbbbbbb0001/pay \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"paid_at":"07/10/2026","amount":95400.00,"payment_method":"débito automático"}'

curl "https://api.ejemplo.co/api/v1/summary?month=10-2026" \
  -H "Authorization: Bearer $TOKEN"
```

## Límites y alcance del MVP

La API del MVP se limita a organizar y recordar obligaciones. No inicia pagos ni movimientos de dinero, no se integra con bancos ni billeteras, no guarda credenciales bancarias ni números completos de tarjetas y no incluye funciones de IA ni de reconocimiento de documentos. Los métodos de pago son solo etiquetas genéricas que describen cómo pagó el usuario por fuera del sistema. Las funciones de importación de facturas, análisis inteligente e integraciones con correo y calendario pertenecen a fases posteriores y no forman parte de esta especificación.

## Relación con otros documentos

- Los requisitos funcionales y no funcionales viven en [los requisitos](../product/requirements.md) y los flujos de usuario en [los casos de uso](../product/use-cases.md).
- El modelo de datos y las políticas de acceso se definen en [la base de datos](./database.md) y las amenazas con sus mitigaciones en [la seguridad](./security.md).
