---
name: design-system-tokens
description: Mantiene el design system de SafePet (docs/design/DESIGN.md) y su traducción a tokens de PrimeReact/PrimeFlex y FullCalendar: colores, tipografía, espaciado, radios, estados y componentes. Úsala al crear o cambiar tokens, al revisar coherencia visual o al preparar el tema del frontend.
---

# Design system y tokens

## Fuente única
`docs/design/DESIGN.md` (YAML de tokens + reglas). Stitch lo consume; el frontend lo traduce. Todo cambio de token se hace aquí primero, en un PR propio.

## Mapeo a implementación (referencia para frontend-engineer)
| Token DESIGN.md | PrimeReact (styled, tema Lara) | FullCalendar |
|---|---|---|
| `primary`, `primary-hover`, `primary-soft` | `--primary-color`, `--primary-600/700`, `--highlight-bg` | `--fc-button-bg-color`, `--fc-today-bg-color` |
| `surface-*`, `text`, `text-muted` | `--surface-ground/card/border`, `--text-color(-secondary)` | `--fc-border-color`, `--fc-page-bg-color` |
| `appointment_types.*` | `Tag` con `style`/clase por tipo | `eventColor` por tipo (`backgroundColor`/`borderColor`) |
| `radius.md` | `--border-radius` | — |
| `font-family` | `--font-family` | heredado |
| `breakpoints` | PrimeFlex `sm/md/lg/xl` | `views` responsivas |

## Reglas
- Contraste verificado (texto ≥ 4.5:1, UI ≥ 3:1) para cada par fondo/texto nuevo; anotar el ratio en el YAML.
- Ningún color como único portador de significado (tipo/estado siempre con texto o icono).
- No introducir componentes visuales que PrimeReact no tenga sin justificar costo de implementación.
- React Bits solo para movimiento decorativo, con `prefers-reduced-motion`.
- Versionar `version:` en el front matter (semver) y registrar cambios en el PR.

## Revisión de coherencia
Comparar mockups/pantallas contra tokens: color fuera de paleta, radios distintos, tamaños tipográficos fuera de escala, espaciado fuera de la escala 4/8/12/16/24/32/48.

## Salida
Diff de `DESIGN.md` + tabla `token | antes | después | contraste | impacto`.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
