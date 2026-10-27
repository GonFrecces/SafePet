---
name: accessibility
description: Revisa y asegura accesibilidad WCAG 2.2 AA en la SPA (agenda pública, formularios, calendario, backoffice) con PrimeReact. Úsala al construir o revisar pantallas.
---

# Accesibilidad (WCAG 2.2 AA)

## Checklist por pantalla
- **Semántica**: landmarks (`header`, `nav`, `main`), un `h1`, jerarquía de encabezados.
- **Teclado**: todo operable con Tab/Shift+Tab/Enter/Espacio/Esc; foco visible; sin trampas de foco; diálogos devuelven el foco al cerrar.
- **Formularios**: `label` asociado a cada campo; errores con `aria-describedby` y texto (no solo color); campos obligatorios indicados.
- **Contraste**: texto ≥ 4.5:1, texto grande y componentes UI ≥ 3:1 (revisar tema PrimeReact).
- **Objetivos táctiles**: ≥ 24×24 px (2.5.8); en móvil apuntar a 44×44.
- **Calendario (FullCalendar)**: alternativa de lista de horarios disponibles navegable por teclado y lector de pantalla; anunciar selección con `aria-live`.
- **Imágenes/iconos**: `alt` o `aria-hidden` si son decorativos; botones solo-icono con `aria-label`.
- **Movimiento**: animaciones de React Bits respetan `prefers-reduced-motion`.
- **Idioma**: `<html lang="es">`.
- **Zoom/reflow**: usable a 320 px de ancho y 200% de zoom.

## Verificación
- Automática: `@axe-core/react` en dev o `vitest-axe`/`jest-axe` en pruebas; Lighthouse accesibilidad.
- Manual: recorrido solo con teclado; lector de pantalla (NVDA/VoiceOver) en flujo de reserva.

## Salida
Hallazgos `criterio WCAG | componente | severidad | corrección`.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
