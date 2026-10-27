---
name: stitch-prompting
description: Redacta, ejecuta e itera prompts de Google Stitch para generar mockups de SafePet coherentes con el design system (móvil y escritorio), y revisa el resultado antes de exportar. Úsala al crear o actualizar pantallas en Stitch.
---

# Prompting en Google Stitch

## Hechos de la herramienta (verificar si pasaron > 3 meses: es Beta)
- Prompts **en inglés**; copy de UI en español entre comillas.
- ≤ 5 pantallas por prompt; estilo compartido dentro del proyecto.
- Design system por `DESIGN.md`; los cambios **no** se propagan solos (seleccionar pantallas → Modify → aplicar).
- Iteración: *Annotate* (un elemento), *Variations* (bloquear color/fuente), *Play* (prototipo).
- Export: Figma, HTML/Tailwind, PNG. MCP: `https://stitch.googleapis.com/mcp` (`generate_screen_from_text`, `edit_screens`, `generate_variants`, `upload_design_md`, `apply_design_system`, `get_screen`).

## Estructura de un prompt de pantalla
1. **Dispositivo y tamaño**: `mobile 390x844` o `desktop 1440x900`.
2. **Contexto**: superficie (pública/backoffice), rol, sucursal activa.
3. **Lista numerada de pantallas** (≤ 5), cada una con: título en español, layout, componentes (nombrados como PrimeReact), datos ficticios concretos, estado destacado (vacío/error/seleccionado).
4. **Restricciones**: "same design system", sin login en lo público, sin elegir veterinario, pago con placeholder de Stripe.

## Reglas
- Concreto > adjetivos: nombra campos, copy y datos ("Luna (Perro) · M. González"), no "modern and clean".
- Un objetivo por iteración; usar Annotate para cambios locales y no regenerar pantallas buenas.
- Primero móvil, después la adaptación a escritorio en un prompt aparte.
- Modo *Fast* para explorar, *Thinking with 3 Pro* para la versión final.
- Registrar en `docs/design/<feature>/prompts.md` cada prompt usado (para reproducibilidad).

## Checklist de revisión
- [ ] Copy en español, datos ficticios.
- [ ] Cada control tiene equivalente PrimeReact; agenda estilo FullCalendar.
- [ ] Tipo de atención con color **y** texto; estados vacío/cargando/error.
- [ ] Contraste AA, foco visible, objetivos ≥ 44px en móvil.
- [ ] Contexto de sucursal visible; nada de otro tenant.
- [ ] Coherencia entre pantallas (shell, radios, tipografía).

## Salida
`prompts.md` + enlaces Stitch/Figma + PNG en `screens/`, con lista de desvíos conocidos frente a `DESIGN.md`.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
