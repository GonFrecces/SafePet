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
  surface-200: "#E2E8F0" # bordes decorativos (cards, separadores); NO para bordes de inputs
  border-input: "#64748B" # slate-500 · borde de inputs/selects/checkbox (4.76:1; WCAG 1.4.11 exige ≥ 3:1)
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

### Tokens → variables PrimeReact (tema Lara, modo styled)

Se aplican como override de variables CSS `:root` sobre `lara-light-teal` (lo implementa [GON-6](https://linear.app/gonfreccess/issue/GON-6/scaffold-del-frontend-react-primereact)). Los nombres de variable son los de PrimeReact 10 (`--primary-color`, `--surface-*`, `--border-radius`…); **[Hipótesis]** verificar contra la versión instalada al hacer el scaffold.

| Token DESIGN.md          | Variable PrimeReact / PrimeFlex                                                                                            | Valor                          |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------- | ------------------------------ |
| `primary`                | `--primary-color`, `--highlight-bg` (texto: `--primary-color-text` = blanco)                                               | `#0F766E`                      |
| `primary-hover`          | `--primary-600`… hover de `Button` (`--primary-color` oscurecido)                                                          | `#115E59`                      |
| `primary-soft`           | `--highlight-bg` (selección), `--primary-100`                                                                              | `#CCFBF1`                      |
| `surface-0`              | `--surface-card`, `--surface-0`                                                                                            | `#FFFFFF`                      |
| `surface-50`             | `--surface-ground`, `--surface-50`                                                                                         | `#F8FAFC`                      |
| `surface-100`            | `--surface-section`, `--surface-100`                                                                                       | `#F1F5F9`                      |
| `surface-200`            | `--surface-border`, `--surface-200`                                                                                        | `#E2E8F0`                      |
| `border-input`           | `--surface-border` solo en `.p-inputtext`, `.p-dropdown`, `.p-checkbox-box`, `.p-radiobutton-box` (override de componente) | `#64748B`                      |
| `text`                   | `--text-color`                                                                                                             | `#0F172A`                      |
| `text-muted`             | `--text-color-secondary`                                                                                                   | `#475569`                      |
| `success`                | `--green-700` (severity `success`)                                                                                         | `#15803D`                      |
| `warning`                | `--orange-700`/`--yellow-700` (severity `warning`; usar el valor del token)                                                | `#B45309`                      |
| `danger`                 | `--red-700` (severity `danger`)                                                                                            | `#B91C1C`                      |
| `info`                   | `--blue-700` (severity `info`)                                                                                             | `#1D4ED8`                      |
| `radius.md` / `lg`       | `--border-radius` (6px) / clase propia en `Card` (12px)                                                                    | `6px` / `12px`                 |
| `typography.font-family` | `--font-family`                                                                                                            | `Inter, system-ui, sans-serif` |
| `elevation.card`         | `--card-shadow`                                                                                                            | `0 1px 3px rgba(15,23,42,.08)` |
| `appointment_types.*`    | `Tag` con `style`/clase `tag-<tipo>` + color de evento FullCalendar (`eventColor`)                                         | ver tabla de tipos             |
| foco                     | `--focus-ring: 0 0 0 2px #FFFFFF, 0 0 0 4px #0F766E`                                                                       | anillo 2px, 5.47:1             |

### Tipos de atención (Tag + evento)

Fondo sólido, texto blanco `#FFFFFF`, **siempre** con etiqueta de texto (y opcionalmente icono). Ningún tipo depende solo del color.

| Tipo          | Etiqueta UI   | Fondo     | Contraste texto blanco |
| ------------- | ------------- | --------- | ---------------------- |
| CONSULTA      | Consulta      | `#1D4ED8` | 6.70:1                 |
| TRATAMIENTO   | Tratamiento   | `#6D28D9` | 7.10:1                 |
| CORTE_DE_PELO | Corte de pelo | `#BE185D` | 6.04:1                 |
| URGENCIA      | Urgencia      | `#B91C1C` | 6.47:1                 |
| PROCEDIMIENTO | Procedimiento | `#B45309` | 5.02:1                 |

### Verificación de contraste (WCAG 2.2 AA)

Herramienta: cálculo propio de luminancia relativa y razón de contraste según la fórmula WCAG 2.x (script Python, 2026-11-03). Umbrales: texto ≥ 4.5:1, componentes de UI y foco ≥ 3:1.

| Par (frente / fondo)                            | Razón       | Umbral  | Resultado                      |
| ----------------------------------------------- | ----------- | ------- | ------------------------------ |
| `text` / `surface-0`                            | 17.85       | 4.5     | ✅                             |
| `text-muted` / `surface-0` · `surface-50`       | 7.58 · 7.24 | 4.5     | ✅                             |
| blanco / `primary` (texto de `Button`)          | 5.47        | 4.5     | ✅                             |
| blanco / `primary-hover`                        | 7.58        | 4.5     | ✅                             |
| `primary` / `primary-soft` (texto en selección) | 4.86        | 4.5     | ✅                             |
| `text` / `primary-soft`                         | 15.84       | 4.5     | ✅                             |
| `success` / `surface-0` · `surface-50`          | 5.02 · 4.79 | 4.5     | ✅                             |
| `warning` / `surface-0` · `surface-50`          | 5.02 · 4.80 | 4.5     | ✅                             |
| `danger` / `surface-0` · `surface-50`           | 6.47 · 6.18 | 4.5     | ✅                             |
| `info` / `surface-0` · `surface-50`             | 6.70 · 6.41 | 4.5     | ✅                             |
| `border-input` / `surface-0` · `surface-50`     | 4.76 · 4.55 | 3       | ✅                             |
| `primary` (anillo de foco) / `surface-0`        | 5.47        | 3       | ✅                             |
| `surface-200` / `surface-0` (borde decorativo)  | 1.23        | 3       | ❌ por eso no se usa en inputs |
| `accent` `#F97316` / `surface-0`                | 2.80        | 3 / 4.5 | ❌ por eso es solo decorativo  |

Hallazgo corregido en este cambio: el borde de input heredado de Lara (`#E2E8F0`) no cumplía 1.4.11; se agrega `border-input`.

**Pendiente** (no verificado): contraste de los estados `disabled`, `:hover` de filas de `DataTable` y colores de eventos FullCalendar sobre cuadrícula; se revisan cuando exista el tema real (GON-6).

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
