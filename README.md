# Shadow-Net Bills

Shadow-Net Bills es una aplicación para organizar y recordar las obligaciones
financieras personales en Colombia: facturas, suscripciones y pagos recurrentes
con recordatorios por correo electrónico, un calendario mensual y un resumen de
gastos. El MVP es manual-primero: sin integraciones bancarias, sin iniciación
de pagos y sin IA/OCR (pospuestos a V1). Está dirigida a personas con múltiples
obligaciones mensuales que hoy dependen de recordatorios dispersos. El estado
actual es planificación: este repositorio contiene solo la especificación.

Estado: Fase de planificación - documentación

Documentación: ver [docs/](docs/).

Nota: todavía NO hay código en este repositorio. No existen backend/,
frontend/ ni infraestructura; solo la especificación en docs/.

## Canonical names

**Canonical names (Waves 2-4 copy VERBATIM - no synonyms):** table `obligations` with column `next_due_date`; API base `/api/v1/obligations`; the 9 fixed categories of §E (Vivienda, Servicios públicos, Telecomunicaciones, Suscripciones, Financiero, Seguros, Educación, Gobierno, Otros); reminder offsets `{3,1,0}` days; generic payment methods (tarjeta, transferencia, efectivo, débito automático, otro).
