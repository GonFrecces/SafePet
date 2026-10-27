---
name: system-design
description: Diseña la estructura de un sistema: módulos y límites, flujos críticos, modelo de datos multi-tenant, integraciones y requisitos no funcionales. Úsala al definir o revisar la arquitectura de SafePet.
---

# Diseño de sistema

## Módulos de referencia (SafePet)
| Módulo | Responsabilidad | Datos propios |
|---|---|---|
| Tenancy | Compañías, sucursales, resolución de tenant | Company, Branch |
| Staff & Auth | Usuarios, roles OWNER/PROFESIONAL/ADMINISTRATIVO | User, Role |
| Scheduling | Disponibilidad por sucursal, reservas públicas, abrir/cerrar cita | Availability, Appointment |
| Clinical | Tutores, mascotas, ficha, atenciones, recetas, tratamientos | Owner, Pet, Encounter, Prescription |
| Billing | Catálogo, cobros, Stripe, webhooks | Product, Payment |
| Notifications | Recordatorios previos a la cita | NotificationJob |

## Procedimiento
1. Límites y dependencias entre módulos (sin ciclos; comunicación por interfaces de `Application`).
2. **Flujos críticos** en secuencia numerada: reserva pública sin login (incluye control de doble reserva: constraint único + transacción), apertura/cierre de cita, cobro con webhook idempotente, envío de recordatorio.
3. **Multi-tenancy.** `TenantId` (+`BranchId`) en entidades de clínica, global query filter, resolución desde usuario autenticado o desde sucursal pública validada.
4. **NFR.** Disponibilidad esperada en free tier (cold starts), latencia, zona horaria (UTC + conversión por sucursal), auditoría, backups.
5. **Puntos de falla** y comportamiento ante cada uno (BD caída, Stripe caído, job de notificación falla).

## Salida
Diagrama textual, tabla de módulos, flujos, decisiones abiertas y riesgos.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
