---
name: SafePet
version: 0.1.0
status: Propuesto
platform: web-responsive (mobile first)
ui_library: PrimeReact (styled mode, tema base Lara, primario personalizado)
colors:
  primary: "#0F766E" # teal-700 · botones, links, foco (5.5:1 sobre blanco)
  primary-hover: "#115E59" # teal-800
  primary-soft: "#CCFBF1" # teal-100 · fondos de selección
  accent: "#F97316" # orange-500 · solo decorativo / ilustración, nunca texto
  surface-0: "#FFFFFF"
  surface-50: "#F8FAFC" # fondo de app
  surface-100: "#F1F5F9"
  surface-200: "#E2E8F0" # bordes
  text: "#0F172A" # slate-900
  text-muted: "#475569" # slate-600 (7.6:1)
  success: "#15803D"
  warning: "#B45309"
  danger: "#B91C1C"
  info: "#1D4ED8"
appointment_types: # color de evento en FullCalendar + Tag
  CONSULTA: "#1D4ED8"
  TRATAMIENTO: "#6D28D9"
  CORTE_DE_PELO: "#BE185D"
  URGENCIA: "#B91C1C"
  PROCEDIMIENTO: "#B45309"
typography:
  font-family: "Inter, system-ui, sans-serif"
  scale:
    {
      h1: 28/36 600,
      h2: 22/30 600,
      h3: 18/26 600,
      body: 16/24 400,
      small: 14/20 400,
      caption: 12/16 500,
    }
radius: { sm: 4px, md: 6px, lg: 12px, pill: 999px }
spacing: [4, 8, 12, 16, 24, 32, 48]
breakpoints: { sm: 576px, md: 768px, lg: 992px, xl: 1200px } # = PrimeFlex
elevation:
  {
    card: "0 1px 3px rgba(15,23,42,.08)",
    overlay: "0 8px 24px rgba(15,23,42,.16)",
  }
touch_target_min: 44px
motion:
  { duration: 150-250ms, easing: ease-out, respects: prefers-reduced-motion }
---

# SafePet — Design System (para Google Stitch y PrimeReact)

## Overview

SaaS multi-tenant para clínicas veterinarias. Dos superficies:

1. **Agenda pública** (cliente anónimo, móvil primero): cálida, simple, 1 tarea por pantalla, wizard de 4 pasos.
2. **Backoffice** (OWNER, PROFESIONAL, ADMINISTRATIVO; escritorio primero, usable en móvil): denso pero limpio, sidebar + topbar con selector de sucursal.

Tono visual: clínico-confiable (teal + blanco + slate), con calidez puntual (ilustraciones/acentos naranjos, esquinas redondeadas 12px en cards). Sin degradados fuertes ni glassmorphism.

## Colors

- `primary` para acciones primarias y estado seleccionado; `primary-soft` para fondos de selección (slot elegido, fila activa).
- `accent` solo en ilustraciones, badges decorativos o iconografía; nunca como color de texto ni único indicador de estado.
- Tipos de atención con color fijo (`appointment_types`) + **siempre** etiqueta de texto (no depender solo del color).
- Modo oscuro: fase 2 (equivalente `lara-dark`), no bloquear el MVP.

## Typography

Inter. Un `h1` por pantalla. Números (montos, horas) con `font-variant-numeric: tabular-nums`.

## Components (mapear SIEMPRE a PrimeReact)

| Patrón                          | Componente PrimeReact                                                               |
| ------------------------------- | ----------------------------------------------------------------------------------- |
| Botón primario/secundario/texto | `Button` (`severity`, `outlined`, `text`)                                           |
| Wizard de reserva               | `Steps` (móvil: indicador "Paso 2 de 4" + `ProgressBar`)                            |
| Selección de tipo de atención   | `SelectButton` o cards seleccionables (`Card` + `RadioButton`)                      |
| Fecha                           | `Calendar` inline (`inline`, `minDate`)                                             |
| Horarios disponibles            | grilla de `Button` outlined / `SelectButton` (chips de hora)                        |
| Agenda backoffice               | `@fullcalendar/react` (timeGridWeek / timeGridDay / listDay)                        |
| Detalle de cita                 | `Sidebar` (drawer derecho) o `Dialog` en móvil a pantalla completa                  |
| Tablas                          | `DataTable` (paginación, filtros, `responsiveLayout="stack"` en móvil)              |
| Formularios                     | `InputText`, `InputNumber`, `Dropdown`, `AutoComplete`, `InputTextarea`, `Calendar` |
| Estados/etiquetas               | `Tag`, `Badge`, `Chip`                                                              |
| Historial clínico               | `Timeline`                                                                          |
| Pestañas de ficha               | `TabView`                                                                           |
| Feedback                        | `Toast`, `Message`, `ConfirmDialog`, `Skeleton`                                     |
| Navegación                      | `Menu`/`PanelMenu` (sidebar), `Menubar`, `Breadcrumb`, `Avatar`                     |
| Pago                            | `Card` + Stripe Payment Element (iframe de Stripe, no inputs propios)               |

React Bits: solo micro-animaciones (contadores de KPI, aparición de la confirmación). Nada de animación en formularios clínicos.

## Layout

- Público: columna única máx. 560px, centrada; CTA fijo inferior en móvil (`position: sticky`).
- Backoffice: sidebar 248px (colapsable a 72px; en móvil `Sidebar` overlay), topbar 56px con selector de sucursal (`Dropdown`), buscador global de mascota/tutor, avatar con rol.
- Grid de 12 columnas (PrimeFlex); gutter 16px móvil / 24px escritorio.

## States (obligatorios en cada pantalla)

Vacío, cargando (`Skeleton`), error (mensaje + reintentar), sin permisos (rol), éxito. Formularios: error inline con texto.

## Domain vocabulary (copy en español)

- Cliente = "tutor"; paciente = "mascota". Tipos: Consulta, Tratamiento, Corte de pelo, Urgencia, Procedimiento.
- Estados de cita **[Hipótesis — confirmar con el dominio]**: Agendada, En atención, Cerrada, Cancelada, No asistió.
- Roles: Dueño (OWNER), Veterinario (PROFESIONAL), Administrativo (ADMINISTRATIVO).
- Moneda **[Hipótesis]**: CLP sin decimales (`$25.000`); ajustar si cambia el mercado.

## Do's and Don'ts

- ✅ Mobile first: diseñar 390px y luego 1440px.
- ✅ Contraste WCAG 2.2 AA (texto ≥ 4.5:1, UI ≥ 3:1), foco visible, objetivos ≥ 44px en móvil.
- ✅ La disponibilidad es de la **clínica/sucursal**: nunca mostrar "elige veterinario" en la agenda pública.
- ✅ Datos ficticios en mockups (nunca datos reales de clientes).
- ❌ No mostrar datos de otra sucursal/tenant en el mismo layout (el selector de sucursal filtra todo).
- ❌ No pedir login/registro al cliente que agenda.
- ❌ No diseñar inputs de tarjeta propios: el pago usa el elemento embebido de Stripe.
