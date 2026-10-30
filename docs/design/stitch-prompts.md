# Prompts para Google Stitch — SafePet

Prompts listos para pegar en [Google Stitch](https://stitch.withgoogle.com/) para generar mockups de SafePet (agenda pública + backoffice), en móvil y escritorio, coherentes con PrimeReact. Design system: [`DESIGN.md`](DESIGN.md).

## 1. Capacidades y límites de Stitch (consultado 2026-10-27)

| Tema          | Estado                                                                                                                                                                                                                                                                                                     | Fuente                                                                                                                                                                    |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Producto      | Google Labs, **Beta**. Gratis con límite de créditos diarios (reset 00:00 UTC; saldo en Settings). Puede cambiar o descontinuarse                                                                                                                                                                          | stitch.withgoogle.com (meta "Stitch (Beta)"); [PyTorchKR, 2026-07](https://discuss.pytorch.kr/t/google-stitch-ui-google-labs-stitch-sdk-mcp-design-md-agent-skills/11094) |
| Idioma        | Interfaz y prompts **solo en inglés** → los prompts van en inglés y el copy de UI se fija entre comillas en español                                                                                                                                                                                        | PyTorchKR                                                                                                                                                                 |
| Modos/modelos | _Thinking with 3 Pro_ (Gemini 3 Pro, candidatos finales), _Redesign_ (Nano Banana Pro), _2.5 Pro_, _Fast_ (wireframes + export Figma). MCP: `GEMINI_3_FLASH` / `GEMINI_3_1_PRO`                                                                                                                            | PyTorchKR                                                                                                                                                                 |
| Generación    | Hasta **5 pantallas por prompt** con estilo compartido; canvas infinito; usa el historial del proyecto; _Agent Manager_ para direcciones en paralelo                                                                                                                                                       | [SFAI Labs, act. 2026-08](https://sfailabs.com/guides/google-stitch-vs-figma)                                                                                             |
| Entradas      | Texto, voz, bocetos/screenshots, imágenes de referencia, URL (para extraer DESIGN.md)                                                                                                                                                                                                                      | SFAI Labs                                                                                                                                                                 |
| Design system | `DESIGN.md` (YAML de tokens + markdown). Se sube o se extrae de URL. **Cambios del design system no se propagan solos**: seleccionar pantallas → _Modify_ → aplicar sistema                                                                                                                                | [Tutorials Dojo](https://tutorialsdojo.com/what-google-stitch-actually-does-and-where-it-falls-short/)                                                                    |
| Iteración     | _Annotate_ (editar un elemento), _Variations_ (3 por defecto; Refine/Explore/Reimagine; bloquear layout/color/fuente/texto)                                                                                                                                                                                | Tutorials Dojo                                                                                                                                                            |
| Prototipo     | _Play_: enlaza pantallas en un flujo clicable y genera las faltantes                                                                                                                                                                                                                                       | SFAI Labs                                                                                                                                                                 |
| Export        | **Figma** (Auto Layout, capas nombradas; requiere limpieza), **HTML/CSS + Tailwind** (zip o portapapeles), React/Flutter según fuente, imagen (PNG)                                                                                                                                                        | Tutorials Dojo, PyTorchKR                                                                                                                                                 |
| MCP / SDK     | Servidor remoto `https://stitch.googleapis.com/mcp` (API key en header `X-Goog-Api-Key` u OAuth). 14 tools: `list_projects`, `get_screen`, `generate_screen_from_text`, `edit_screens`, `generate_variants`, `upload_design_md`, `apply_design_system`… Skills oficiales: `google-labs-code/stitch-skills` | PyTorchKR                                                                                                                                                                 |
| Límites       | Un solo usuario (sin colaboración); control fino de spacing/responsive inferior a Figma; el código exportado es **Tailwind, no PrimeReact**                                                                                                                                                                | SFAI Labs, Tutorials Dojo                                                                                                                                                 |

> **Hipótesis a verificar en la cuenta:** cupo exacto de créditos y si el modo elegido permite export a Figma (fuentes antiguas indican que el modo experimental no lo permitía).
>
> **Implicación para SafePet:** Stitch produce **referencia visual y de flujo**. El HTML/Tailwind exportado **no** se copia al frontend; el Frontend Engineer implementa con PrimeReact usando la spec de handoff.

## 2. Cómo usarlos

1. Crear proyecto **"SafePet"** en Stitch. Elegir **App (mobile)** para lotes móviles y **Web** para escritorio (o indicar el tamaño en el prompt).
2. Subir `docs/design/DESIGN.md` como design system del proyecto (o pegar el **Prompt 0**).
3. Ejecutar los lotes en orden (cada uno ≤ 5 pantallas). Modo recomendado: _Fast_ para explorar, _Thinking with 3 Pro_ para la versión final.
4. Revisar con la checklist de §5 y aplicar los prompts de iteración (§4).
5. Exportar (§6) y entregar al agente **UI/UX Designer** para la spec de handoff.

---

## 3. Prompts

### Prompt 0 — Contexto y design system (pegar primero)

```text
You are designing "SafePet", a responsive multi-tenant SaaS web app for veterinary clinics. All UI copy must be in Spanish (Latin America). Use fictional sample data only.

Two surfaces:
1) Public booking (anonymous pet owner, no login, mobile first): book an appointment in an available time slot of a clinic BRANCH. Availability belongs to the branch, not to a vet; the owner never picks a vet.
2) Back-office for clinic staff (desktop first, usable on mobile) with roles: Dueño (owner of the company, sees only their own branches), Veterinario (attends patients, opens and closes their own appointments, clinical record, prescriptions, treatments, grooming), Administrativo (schedule, availability, shifts, product sales, charging appointments).

Appointment types (fixed color + always a text label): Consulta #1D4ED8, Tratamiento #6D28D9, Corte de pelo #BE185D, Urgencia #B91C1C, Procedimiento #B45309.

Design system (must stay consistent across every screen):
- Look & feel: PrimeReact "Lara" theme style. Clean, trustworthy clinical look with warmth. White surfaces, app background #F8FAFC, borders #E2E8F0, text #0F172A, muted text #475569.
- Primary teal #0F766E (hover #115E59, soft selection background #CCFBF1). Decorative accent orange #F97316 only for illustrations/icons, never for text.
- Semantic: success #15803D, warning #B45309, danger #B91C1C, info #1D4ED8.
- Font Inter. H1 28px/600, H2 22px/600, H3 18px/600, body 16px, small 14px. Tabular numbers for times and money.
- Radius 6px for inputs/buttons, 12px for cards. Spacing scale 4/8/12/16/24/32/48. Soft card shadow.
- Components should look like PrimeReact: Button, Steps, Calendar (inline), SelectButton, DataTable, Dropdown, InputText, Tag, Badge, Timeline, TabView, Sidebar drawer, Dialog, Toast, Skeleton. The back-office agenda looks like FullCalendar (time grid week/day).
- Accessibility WCAG 2.2 AA: contrast >= 4.5:1, visible focus ring, touch targets >= 44px on mobile, never color as the only signal.
- No gradients-heavy hero, no glassmorphism, no stock-photo clutter. Friendly line illustrations of cats and dogs are allowed in empty states and confirmation.

Acknowledge this system and wait for screen requests.
```

### Lote A — Agenda pública, móvil (390×844) · 5 pantallas

```text
Generate 5 mobile screens (390x844) for the SafePet public booking flow, using the established design system. Clinic: "Clínica Veterinaria Patitas" (fictional). Header with clinic logo placeholder and name; no login anywhere. A 4-step indicator "Paso X de 4" with a thin progress bar. Sticky bottom primary button.

1. "Elige sucursal": clinic hero card (logo, short description, rating hidden), list of 3 branch cards (name, address, distance, "Abierto hoy 09:00–19:00" tag), search input "Buscar por comuna". CTA "Continuar".
2. "¿Qué necesita tu mascota?" (Paso 1): 5 selectable cards with icon, title and duration: Consulta (30 min), Tratamiento (30 min), Corte de pelo (60 min), Urgencia (show warning message "Si es una emergencia vital, llama al +56 2 0000 0000"), Procedimiento (90 min). Selected card uses teal border and #CCFBF1 background.
3. "Elige fecha y hora" (Paso 2): inline month calendar with unavailable days disabled, then a grid of time chips grouped "Mañana" / "Tarde" (e.g. 09:00, 09:30, 10:00…); one chip selected; note "Te atenderá cualquier veterinario disponible de la sucursal". Empty state for a day with no slots: "Sin horas este día" + "Ver próximo día disponible".
4. "Tus datos" (Paso 3): form with sections "Tutor" (nombre, email, teléfono with +56 prefix) and "Mascota" (nombre, especie Perro/Gato/Otro as SelectButton, raza optional, edad aproximada), textarea "Motivo de la visita", checkbox "Acepto recibir un recordatorio por email/SMS" and privacy link. Show one field with an inline validation error in text. Summary card at top: Consulta · Sucursal Providencia · Jue 30 oct · 10:30.
5. "¡Hora agendada!" (Paso 4): success illustration of a cat and dog, booking summary card with code "SP-4F7K2", buttons "Agregar a mi calendario" and "Cancelar o reagendar" (secondary), info message "Te avisaremos 30 minutos antes".
```

### Lote B — Agenda pública, escritorio (1440×900) · 3 pantallas

```text
Adapt the public booking flow to desktop web (1440x900), same design system and same copy. Centered content max-width 1040px, two columns: left = current step, right = sticky "Resumen de tu reserva" card that fills progressively (sucursal, tipo de atención, fecha y hora, mascota). Horizontal PrimeReact-style Steps at top: Atención · Fecha y hora · Tus datos · Confirmación.

1. Step "Atención" with the 5 appointment type cards in a 3-column grid and the branch already chosen (link "Cambiar sucursal").
2. Step "Fecha y hora": inline calendar on the left half, time chips grouped Mañana/Tarde on the right half; one chip selected.
3. Step "Confirmación": success state with summary, code "SP-4F7K2", "Agregar a mi calendario", "Cancelar o reagendar".
```

### Lote C — Backoffice clínico, escritorio (1440×900) · 5 pantallas

```text
Generate 5 desktop back-office screens (1440x900) for SafePet, same design system. Shell: left sidebar 248px (logo, items: Agenda, Pacientes, Atenciones, Recetas, Caja y ventas, Equipo y turnos, Sucursales, Reportes, Configuración; items depend on role), top bar 56px with branch selector dropdown "Sucursal Providencia", global search "Buscar mascota o tutor", notifications bell and avatar "Dra. Camila Rojas · Veterinario".

1. "Iniciar sesión": staff-only login card (email, contraseña, "¿Olvidaste tu contraseña?"), left half with brand illustration. Note "¿Eres cliente? Agenda tu hora aquí" link.
2. "Agenda": FullCalendar-style week time grid 08:00–20:00 for the selected branch, events colored by appointment type with text label, pet name and owner (e.g. "Consulta · Luna (Perro) · M. González"). Toolbar: Hoy, < >, Día/Semana/Lista, filters by type and by state, button "Nueva cita". Right column mini-list "Próximas de hoy". Legend of the 5 types.
3. Appointment detail drawer (right Sidebar 480px) over the agenda: pet + owner header, type Tag, state Tag "Agendada", time, reason, contact, actions: "Abrir atención" (primary), "Reagendar", "Cancelar", "Marcar no asistió". Show audit line "Agendada online · 27 oct 18:42".
4. "Atención en curso": state banner "En atención · iniciada 10:32 por Dra. Camila Rojas". Left: patient card (photo placeholder, especie, raza, edad, peso, alergias tag in danger color). Center TabView: Anamnesis, Examen físico (peso, temperatura, FC, FR inputs), Diagnóstico, Tratamiento, Receta, Notas. Footer actions: "Guardar borrador", "Emitir receta", "Cerrar atención" (primary, opens confirm dialog with charge summary).
5. "Ficha clínica" of a pet: header with pet and owner, tabs Historial / Vacunas / Recetas / Tratamientos / Archivos; Historial as vertical Timeline of past attentions with type tag, vet, date, diagnosis summary; note badge "Ficha de Clínica Veterinaria Patitas" (record belongs to the clinic).
```

### Lote D — Gestión y caja, escritorio (1440×900) · 5 pantallas

```text
Generate 5 desktop management screens (1440x900), same shell and design system.

1. "Emitir receta": form with patient header, repeatable medication rows (medicamento AutoComplete, dosis, frecuencia, duración, vía), indicaciones textarea; right side A4 PDF preview with clinic letterhead and vet signature placeholder. Actions "Guardar" / "Descargar PDF" / "Enviar al tutor".
2. "Cobrar atención": left the itemized charge (Consulta $25.000, Vacuna antirrábica $15.000, descuento), totals with tabular numbers; right a payment panel with method SelectButton (Tarjeta / Efectivo / Transferencia); for Tarjeta show an embedded Stripe Payment Element placeholder box labeled "Pago seguro con Stripe" (do not draw custom card inputs). Success Toast "Pago registrado".
3. "Caja y ventas" (POS): product grid with search and categories (Alimentos, Antiparasitarios, Accesorios), cart panel on the right with quantities, total and "Cobrar". Stock badge "Quedan 3".
4. "Equipo y turnos": DataTable of staff (nombre, rol Tag, sucursal, estado), and a weekly shift grid per vet; button "Invitar miembro".
5. "Disponibilidad de la sucursal": weekly opening hours editor per weekday (toggle + time ranges), slot duration per appointment type, simultaneous capacity (number of boxes), exceptions/holidays list with Calendar. Info message "Los clientes ven horas según la disponibilidad de la sucursal, no de cada veterinario".
```

### Lote E — Backoffice móvil (390×844) · 4 pantallas

```text
Generate 4 mobile back-office screens (390x844) for a vet using SafePet on the phone, same design system. Top bar with hamburger (sidebar as overlay), branch selector compact, avatar.

1. "Mi agenda de hoy": FullCalendar list-day style, cards per appointment with time, type Tag, pet, owner, state; swipe hint; FAB "Nueva cita".
2. "Atención en curso" mobile: sticky header with pet and timer, accordion sections (Examen físico, Diagnóstico, Tratamiento, Receta), sticky bottom "Cerrar atención".
3. "Ficha clínica" mobile: pet header, horizontal scrollable tabs, Timeline of attentions.
4. "Cobrar atención" mobile: itemized total, method selector, Stripe Payment Element placeholder, primary "Cobrar $40.000".
```

### Lote F — Dueño (OWNER), escritorio · 2 pantallas

```text
Generate 2 desktop screens (1440x900) for the role Dueño, same shell. The branch selector includes "Todas mis sucursales".

1. "Resumen": KPI cards (Citas hoy, Ingresos del mes, Tasa de no asistencia, Ocupación), bar chart of appointments by type, line chart of revenue by branch, table "Sucursales" with occupancy. Only this company's branches.
2. "Sucursales": list/DataTable of branches (nombre, dirección, horario, veterinarios, estado) with "Nueva sucursal" and a drawer to edit branch data and public booking link "safepet.app/patitas/providencia" with copy button.
```

---

## 4. Prompts de iteración

Usar _Annotate_ para cambios puntuales y _Variations_ (bloquear **color** y **fuente**) para explorar layout.

| Objetivo             | Prompt                                                                                                                                                                                              |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Consistencia         | `Apply the SafePet design system to all selected screens: same sidebar, top bar, radius, spacing, colors and typography. Fix any screen that deviates.`                                             |
| Alinear a PrimeReact | `Make every control look like a standard PrimeReact Lara component (Button, Dropdown, Calendar, DataTable, Tag, TabView, Sidebar). Remove custom widgets that PrimeReact does not provide.`         |
| Accesibilidad        | `Audit contrast to WCAG 2.2 AA: body text >= 4.5:1, UI borders and icons >= 3:1. Add visible focus rings, text labels for every icon-only button, and make all mobile touch targets at least 44px.` |
| Estados              | `For this screen create variants: loading (skeletons), empty state with a friendly pet illustration, error with retry, and no-permission for role Administrativo.`                                  |
| Densidad backoffice  | `Increase information density for desktop staff use: 14px table text, compact rows, keep 16px for forms. Do not reduce contrast.`                                                                   |
| Copy                 | `Review all copy: Spanish (Latin America), use "tutor" and "mascota", short action verbs on buttons, dates like "jue 30 oct · 10:30", money like "$25.000".`                                        |
| Responsive           | `Create the 390px mobile version of this desktop screen: sidebar becomes overlay, tables become stacked cards, primary action sticky at the bottom.`                                                |
| Multi-tenant         | `Make the current branch and company context always visible in the top bar; nothing on screen may show data from another branch unless "Todas mis sucursales" is selected by role Dueño.`           |
| Prototipo            | `Link the public booking screens in order with Play mode: Elige sucursal → Atención → Fecha y hora → Tus datos → Confirmación.`                                                                     |
| Modo oscuro (fase 2) | `Create a dark theme variant equivalent to PrimeReact lara-dark-teal, keeping AA contrast.`                                                                                                         |

## 5. Checklist de revisión (antes de exportar)

- [ ] Copy 100% en español; datos ficticios; ningún dato real.
- [ ] Agenda pública: sin login, sin elegir veterinario, 4 pasos, CTA sticky, estado sin horas.
- [ ] Tipos de atención con color **y** texto; estados de cita visibles.
- [ ] Contraste AA, foco visible, objetivos ≥ 44px en móvil.
- [ ] Cada control tiene equivalente PrimeReact (tabla de `DESIGN.md`); calendario estilo FullCalendar.
- [ ] Contexto de sucursal visible en el backoffice; nada de otro tenant.
- [ ] Pago con placeholder de Stripe, sin inputs de tarjeta propios.
- [ ] Móvil (390) y escritorio (1440) para cada flujo crítico.

## 6. Exportación y entrega

1. **Figma** (si el modo lo permite): exportar el proyecto; nombrar frames `SP-<lote><n>-<pantalla>-<mobile|desktop>`.
2. **PNG** de cada pantalla → `docs/design/<feature>/screens/`.
3. **HTML/Tailwind** solo como referencia → `docs/design/<feature>/reference/` (no se importa al frontend).
4. Enlace del proyecto Stitch + Figma en el issue de diseño de Linear.
5. **Wanda** (UX/UI Designer) redacta `docs/design/<feature>/spec.md` y crea el issue para `agent:shuri-frontend-engineer` (ver `docs/equipo-agentes.md` → flujo de diseño).

> Si se usa el MCP de Stitch desde Claude Code/Multica, la API key va **solo** como variable de entorno del runtime (p. ej. `STITCH_API_KEY`), nunca en el repo, issues ni prompts (`CLAUDE.md` §6).
