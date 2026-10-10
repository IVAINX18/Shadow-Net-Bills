# Modelo de datos MVP de Shadow-Net Bills

## Alcance del documento

Este documento describe el modelo de datos del MVP de Shadow-Net Bills: las 6 tablas del MVP, sus columnas, relaciones, índices, restricciones y políticas de seguridad a nivel de fila (RLS) en PostgreSQL gestionado por Supabase. El público objetivo es el equipo de desarrollo que va a implementar la API y el esquema de base de datos. Todo el contenido aplica al MVP manual-primero: no incluye IA ni OCR, no mueve dinero y no almacena credenciales bancarias ni números completos de tarjetas.

Las decisiones de producto que condicionan este diseño están aprobadas: MVP manual-primero de 8 semanas, 6 tablas (profiles, categories, obligations, payments, notification_log, app_settings), 9 categorías fijas (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros), métodos de pago genéricos (tarjeta, transferencia, efectivo, débito automático, otro) y recordatorios por correo a 3, 1 y 0 días del vencimiento. La tabla central es `obligations` con la columna `next_due_date` para la recurrencia.

## Convenciones generales

Todas las tablas del MVP siguen estas convenciones, sin excepción:

- Zona horaria: la aplicación trabaja en `America/Bogota`. Las columnas de fecha y hora usan `TIMESTAMPTZ` y la API convierte a `America/Bogota` antes de mostrar. El ajuste por defecto de cada usuario vive en `profiles.timezone` y `app_settings.timezone`, con valor por defecto `'America/Bogota'`.
- Borrado en cascada: las claves foráneas dependientes usan `ON DELETE CASCADE`. Si se elimina un usuario de `auth.users`, se eliminan sus filas en `profiles`, `categories` propias, `obligations`, `payments`, `notification_log` y `app_settings`. Si se elimina una obligación, se eliminan sus pagos y su historial de notificaciones.
- Claves primarias: `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` en todas las tablas.
- Propiedad: cada fila de usuario lleva `user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE`, que es la columna que usan las políticas RLS. La única excepción es `categories`, donde `user_id` es nullable para las filas de sistema (ver sección de categorías).
- Auditoría mínima: `created_at TIMESTAMPTZ NOT NULL DEFAULT now()` en todas las tablas; `updated_at TIMESTAMPTZ NOT NULL DEFAULT now()` en las tablas editables (profiles, obligations, app_settings).
- Moneda: `currency TEXT NOT NULL DEFAULT 'COP'`. Los montos se formatean como `$20.000` en la interfaz.
- Montos: `DECIMAL(12,2)` con restricción `CHECK (amount >= 0)`. El sistema nunca guarda saldos negativos.
- Nombres de tablas y columnas en inglés con minúsculas y guion bajo, coherentes con la API base `/api/v1/obligations`.

## Diagrama entidad-relación

Diagrama ASCII del MVP. Cada línea es una relación con borrado en cascada. La tabla `categories` tiene dos tipos de filas: sistema (`user_id` nulo) y de usuario.

```
auth.users (Supabase Auth)
    |
    | 1:N  ON DELETE CASCADE
    v
profiles (1:1 por usuario, user_id único)
    |
    +---> categories (N:1 por usuario; + filas de sistema con user_id NULL)
    |         |
    |         | 1:N  ON DELETE CASCADE (category_id)
    |         v
    +---> obligations (N:1 por usuario, N:1 por categoría)
    |         |
    |         +---> payments (N:1, obligation_id ON DELETE CASCADE)
    |         |
    |         +---> notification_log (N:1, obligation_id ON DELETE CASCADE)
    |
    +---> app_settings (1:1 por usuario, user_id único)
```

Lectura del diagrama: un usuario posee muchas categorías propias, muchas obligaciones, muchos pagos, muchas entradas de notificación y una fila de ajustes. Cada obligación pertenece a una categoría y genera cero o más pagos y cero o más entradas en `notification_log`. Las filas de sistema de `categories` no pertenecen a ningún usuario y son legibles por todos los usuarios autenticados.

## Tablas del MVP

El MVP tiene exactamente 6 tablas. No se diseña ninguna otra tabla en esta fase. Las tablas previstas para V1 y V2 (subscriptions, installments, documents, extracted_data, audit_logs) se listan solo como roadmap en la última sección, sin DDL.

### profiles

Perfil extendido del usuario autenticado con Supabase Auth. Relación 1:1 con `auth.users` mediante `user_id`. Guarda el nombre visible y la zona horaria. No guarda contraseñas ni datos de tarjetas: la autenticación la gestiona Supabase Auth y Bills solo conserva el correo y el nombre.

```sql
CREATE TABLE profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE REFERENCES auth.users(id) ON DELETE CASCADE,
    email TEXT NOT NULL,
    full_name TEXT,
    timezone TEXT NOT NULL DEFAULT 'America/Bogota',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE
);

CREATE INDEX idx_profiles_user_id ON profiles(user_id);

ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

CREATE POLICY profiles_owner_policy ON profiles
    FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

Notas: `user_id` es único porque cada usuario tiene un solo perfil. `email` es `NOT NULL` porque se necesita para los recordatorios. La política RLS limita toda operación al dueño de la fila mediante `auth.uid() = user_id`.

### categories

Catálogo de categorías. Contiene las 9 categorías fijas del sistema (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros) como filas de sistema con `user_id` nulo, más las categorías personalizadas que cree cada usuario con su `user_id` como dueño.

```sql
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    is_system BOOLEAN NOT NULL DEFAULT false,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE,
    CHECK (name <> ''),
    CHECK ((is_system = true AND user_id IS NULL) OR (is_system = false AND user_id IS NOT NULL))
);

CREATE INDEX idx_categories_user_id ON categories(user_id);
CREATE INDEX idx_categories_name ON categories(name);

ALTER TABLE categories ENABLE ROW LEVEL SECURITY;

CREATE POLICY categories_read_policy ON categories
    FOR SELECT
    USING (user_id IS NULL OR auth.uid() = user_id);

CREATE POLICY categories_owner_write_policy ON categories
    FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

Convención de propiedad: las filas de sistema (`user_id` nulo, `is_system = true`) representan las 9 categorías fijas y las inserta la migración inicial; ningún usuario puede editarlas ni borrarlas desde la API. Las filas de usuario (`user_id` = dueño) las crea y administra cada usuario. La política RLS de lectura permite leer las filas de sistema a todo usuario autenticado y las propias al dueño; la política de escritura solo permite operar sobre filas donde `auth.uid() = user_id`, por lo que las filas de sistema quedan protegidas.

### obligations

Tabla central del MVP. Representa cada obligación financiera: nombre, monto en COP, categoría, vencimiento, frecuencia de recurrencia, método de pago genérico, notas y configuración de recordatorios. La columna `next_due_date` sostiene la recurrencia: al marcar como pagada una obligación recurrente, el sistema genera el siguiente vencimiento.

```sql
CREATE TABLE obligations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    category_id UUID REFERENCES categories(id) ON DELETE CASCADE,
    name TEXT NOT NULL,
    amount DECIMAL(12,2) NOT NULL,
    currency TEXT NOT NULL DEFAULT 'COP',
    frequency TEXT NOT NULL DEFAULT 'one_time',
    due_date DATE NOT NULL,
    next_due_date DATE,
    payment_method TEXT NOT NULL DEFAULT 'otro',
    notes TEXT,
    reminder_enabled BOOLEAN NOT NULL DEFAULT true,
    reminder_days INT[] NOT NULL DEFAULT '{3,1,0}',
    status TEXT NOT NULL DEFAULT 'pendiente',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE,
    FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE,
    CHECK (amount >= 0),
    CHECK (frequency IN ('one_time', 'weekly', 'monthly', 'quarterly', 'yearly')),
    CHECK (payment_method IN ('tarjeta', 'transferencia', 'efectivo', 'débito automático', 'otro')),
    CHECK (status IN ('pendiente', 'pagada', 'vencida', 'cancelada')),
    CHECK (name <> '')
);

CREATE INDEX idx_obligations_user_id ON obligations(user_id);
CREATE INDEX idx_obligations_due_date ON obligations(user_id, due_date);
CREATE INDEX idx_obligations_status ON obligations(user_id, status);
CREATE INDEX idx_obligations_next_due_date ON obligations(user_id, next_due_date) WHERE status = 'pendiente';

ALTER TABLE obligations ENABLE ROW LEVEL SECURITY;

CREATE POLICY obligations_owner_policy ON obligations
    FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

Notas: `amount` usa `DECIMAL(12,2)` y el `CHECK` impide montos negativos. `frequency` admite `one_time`, `weekly`, `monthly`, `quarterly` y `yearly`. `payment_method` es un enum genérico (tarjeta, transferencia, efectivo, débito automático, otro): describe cómo pagó el usuario, nunca inicia un pago ni guarda números de tarjeta. `reminder_days` vale `{3,1,0}` por defecto. Los índices por `user_id`, `due_date`, `status` y `next_due_date` aceleran el tablero, el calendario mensual y el trabajo del scheduler de recordatorios.

### payments

Historial de pagos. Cada vez que el usuario marca una obligación como pagada, la API inserta una fila aquí con el monto, la fecha de pago y el método usado. Una obligación puede tener muchos pagos a lo largo de sus recurrencias.

```sql
CREATE TABLE payments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    obligation_id UUID NOT NULL REFERENCES obligations(id) ON DELETE CASCADE,
    amount DECIMAL(12,2) NOT NULL,
    paid_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    payment_method TEXT NOT NULL DEFAULT 'otro',
    notes TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE,
    FOREIGN KEY (obligation_id) REFERENCES obligations(id) ON DELETE CASCADE,
    CHECK (amount >= 0),
    CHECK (payment_method IN ('tarjeta', 'transferencia', 'efectivo', 'débito automático', 'otro'))
);

CREATE INDEX idx_payments_user_id ON payments(user_id);
CREATE INDEX idx_payments_obligation_id ON payments(obligation_id);
CREATE INDEX idx_payments_paid_at ON payments(user_id, paid_at);

ALTER TABLE payments ENABLE ROW LEVEL SECURITY;

CREATE POLICY payments_owner_policy ON payments
    FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

Notas: `obligation_id` referencia a `obligations` con `ON DELETE CASCADE`: al borrar una obligación se borra su historial. El índice por `obligation_id` acelera el detalle de la obligación y el índice por `paid_at` acelera el resumen mensual. No se guarda ningún dato de tarjeta, solo el método genérico.

### notification_log

Registro de recordatorios enviados por correo. Cada fila representa un envío para una obligación, un tipo de aviso (3 días antes, 1 día antes, el mismo día) y una fecha de envío. La clave única sobre la terna garantiza la idempotencia: el scheduler puede reintentar sin riesgo de reenviar el mismo aviso dos veces.

```sql
CREATE TABLE notification_log (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE,
    obligation_id UUID NOT NULL REFERENCES obligations(id) ON DELETE CASCADE,
    tipo TEXT NOT NULL,
    fecha_envio DATE NOT NULL,
    sent_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    status TEXT NOT NULL DEFAULT 'enviada',
    error TEXT,
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE,
    FOREIGN KEY (obligation_id) REFERENCES obligations(id) ON DELETE CASCADE,
    UNIQUE (obligation_id, tipo, fecha_envio),
    CHECK (tipo IN ('recordatorio_3d', 'recordatorio_1d', 'recordatorio_0d', 'resumen_mensual')),
    CHECK (status IN ('enviada', 'fallida', 'omitida'))
);

CREATE INDEX idx_notification_log_user_id ON notification_log(user_id);
CREATE INDEX idx_notification_log_obligation_id ON notification_log(obligation_id);
CREATE INDEX idx_notification_log_fecha ON notification_log(obligation_id, fecha_envio);

ALTER TABLE notification_log ENABLE ROW LEVEL SECURITY;

CREATE POLICY notification_log_owner_policy ON notification_log
    FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

Idempotencia: la restricción `UNIQUE (obligation_id, tipo, fecha_envio)` es la clave única de idempotencia (obligation_id más tipo más fecha de envío). Antes de enviar, el worker inserta o verifica esta fila dentro de una transacción; si ya existe, omite el envío. Así un reintento del scheduler o una doble ejecución nunca duplica correos.

### app_settings

Ajustes por usuario. Relación 1:1 con `auth.users` mediante `user_id` único. Guarda la zona horaria, el interruptor global de recordatorios, los desplazamientos de aviso y el idioma.

```sql
CREATE TABLE app_settings (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE REFERENCES auth.users(id) ON DELETE CASCADE,
    timezone TEXT NOT NULL DEFAULT 'America/Bogota',
    email_reminders_enabled BOOLEAN NOT NULL DEFAULT true,
    reminder_offsets INT[] NOT NULL DEFAULT '{3,1,0}',
    language TEXT NOT NULL DEFAULT 'es-CO',
    created_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT now(),
    FOREIGN KEY (user_id) REFERENCES auth.users(id) ON DELETE CASCADE,
    CHECK (language IN ('es-CO', 'es', 'en'))
);

CREATE INDEX idx_app_settings_user_id ON app_settings(user_id);

ALTER TABLE app_settings ENABLE ROW LEVEL SECURITY;

CREATE POLICY app_settings_owner_policy ON app_settings
    FOR ALL
    USING (auth.uid() = user_id)
    WITH CHECK (auth.uid() = user_id);
```

Notas: `reminder_offsets` replica el `{3,1,0}` global y permite al usuario desactivar algún aviso sin tocar cada obligación. Se crea una fila por usuario en el registro. La política RLS restringe todo acceso al dueño con `auth.uid() = user_id`.

## Índices y rendimiento

Los índices del MVP cubren los accesos calientes: tableros y listados filtran por `user_id` más `due_date` o `status`; el calendario filtra por `user_id` más `due_date`; el scheduler busca pendientes por `user_id` más `next_due_date`; el detalle y el resumen usan `obligation_id` y `paid_at`. La lista completa de columnas indexadas incluye `user_id`, `due_date`, `status`, `next_due_date`, `obligation_id` y `paid_at`, cada una con su `CREATE INDEX` en la sección de su tabla. Todos los índices compuestos lideran con `user_id` para que cada consulta quede particionada por dueño desde el planificador. No se introduce ninguna extensión ni índice especializado en el MVP.

## Seguridad a nivel de fila (RLS)

Cada una de las 6 tablas declara `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` y al menos una `CREATE POLICY` que compara `auth.uid() = user_id`. La API valida el JWT de Supabase en cada petición y ejecuta las consultas con el rol del usuario, de modo que RLS nunca se omite y la clave `service_role` nunca se usa para lecturas con alcance de usuario. El frontend nunca accede directo a la base: toda lectura y escritura pasa por la API. El resumen de políticas es: `profiles_owner_policy` (profiles), `categories_read_policy` más `categories_owner_write_policy` (categories, donde la lectura admite filas de sistema con `user_id IS NULL`), `obligations_owner_policy` (obligations), `payments_owner_policy` (payments), `notification_log_owner_policy` (notification_log) y `app_settings_owner_policy` (app_settings). En total hay 7 políticas `CREATE POLICY` para 6 tablas.

## Estados y transiciones de obligations

La columna `status` admite cuatro valores: `pendiente`, `pagada`, `vencida` y `cancelada`. Las transiciones permitidas son:

- `pendiente` hacia `pagada`: el usuario marca la obligación como pagada; la API inserta la fila en `payments`. Si `frequency` es distinta de `one_time`, calcula `next_due_date` y crea la siguiente ocurrencia en `pendiente`.
- `pendiente` hacia `vencida`: transición automática del sistema cuando `due_date` o `next_due_date` queda antes de hoy en `America/Bogota` y no hay pago registrado.
- `pendiente` hacia `cancelada`: el usuario cancela una obligación futura; deja de generar recordatorios y recurrencias.
- `vencida` hacia `pagada`: el usuario paga tarde; se registra el pago con su `paid_at` real.
- `vencida` hacia `cancelada`: el usuario descarta una obligación vencida que ya no va a pagar.
- `pagada` es terminal para esa ocurrencia: no vuelve a `pendiente`. La recurrencia continúa en la siguiente ocurrencia creada.
- `cancelada` es terminal: no genera más vencimientos ni avisos.

La fecha de referencia para vencimientos y avisos es el día calendario en `America/Bogota`. Los avisos se programan a 3, 1 y 0 días según `reminder_days` y `reminder_offsets`, y cada envío queda registrado en `notification_log` bajo la clave de idempotencia.

## Evolución futura (solo roadmap, sin diseño)

Las siguientes tablas están previstas para fases posteriores y se mencionan únicamente como roadmap: no tienen DDL, columnas ni políticas en este documento. La tabla subscriptions sostendrá el módulo de suscripciones en V1 (la suscripción dejará de modelarse como obligación recurrente de categoría Suscripciones). La tabla installments sostendrá el módulo de compras a cuotas en V1 (dejará de modelarse como obligación de categoría Financiero con notas de cuota). La tabla documents guardará los metadatos de facturas subidas para OCR en V1. La tabla extracted_data guardará los campos extraídos por OCR con su confianza y estado de confirmación humana. La tabla audit_logs registrará eventos de auditoría y seguridad. Cualquier diseño de estas tablas pertenece a su fase y a su propio documento, no al MVP.
