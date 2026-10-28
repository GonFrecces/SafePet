---
name: design-handoff
description: Convierte mockups aprobados de Stitch/Figma en una spec implementable (docs/design/<feature>/spec.md) y crea el issue para frontend-engineer con alcance, exclusiones, criterios de aceptación y dependencias de API. Úsala cuando un diseño esté aprobado y deba pasar a implementación.
---

# Handoff de diseño → Frontend

## Precondiciones
- Issue de diseño con aprobación humana explícita (comentario o estado *Aprobado*).
- PNG de móvil y escritorio en `docs/design/<feature>/screens/`.

## `spec.md` (una por feature)
1. **Resumen y roles** que usan la pantalla; ruta propuesta (`/agendar/:clinic/:branch`, `/app/agenda`…).
2. **Pantallas**: por cada una, imagen, propósito, layout móvil/escritorio, breakpoints.
3. **Componentes**: tabla `elemento | componente PrimeReact/FullCalendar | props clave | notas`.
4. **Estados**: vacío, cargando, error, sin permisos, éxito; validaciones con mensajes exactos.
5. **Copy**: textos exactos en español (botones, errores, toasts).
6. **Datos**: campos que la UI muestra/envía → endpoint o contrato esperado; marcar **[Contrato pendiente]** si no existe.
7. **Accesibilidad**: orden de foco, `aria-*`, alternativa al calendario, anuncios `aria-live`.
8. **Fuera de alcance** y **desvíos** frente a `DESIGN.md`.

## Issue para `agent:frontend-engineer` (plantilla)
```
Título: feat(<feature>): implementar <pantalla/flujo> según diseño
Labels: agent:frontend-engineer, Frontend
Bloqueado por: <issue de diseño>  ·  Relacionado: <issue de contrato API si aplica>
Diseño: docs/design/<feature>/spec.md · Stitch/Figma: <enlaces>
Alcance: pantallas X, Y (móvil + escritorio), estados listados en la spec.
Exclusiones: lógica de negocio del backend, endpoints no existentes (usar MSW), modo oscuro.
Criterios de aceptación (dado/cuando/entonces):
- Dado 390px y 1440px, cuando se abre <pantalla>, entonces coincide con el mockup (layout, copy, componentes PrimeReact de la spec).
- Dado <estado vacío/error>, cuando ocurre, entonces se muestra el mensaje exacto de la spec.
- Dado teclado únicamente, cuando se completa el flujo, entonces es operable con foco visible (axe sin violaciones serias).
- Pruebas: npm run lint, npm test (Vitest + Testing Library), npm run build.
```

## Reglas
- Una feature = un issue de implementación (pequeño, revisable); dividir si supera ~5 pantallas.
- El HTML/Tailwind de Stitch se enlaza como referencia, nunca como base de código.
- Si falta un endpoint, no se inventa el contrato: se crea/enlaza issue para `solution-architect`/`backend-engineer`.
- Tras la implementación, el diseñador revisa el PR contra la spec (comentario, no aprobación de merge).

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
