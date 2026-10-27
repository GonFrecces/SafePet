---
name: frontend-architecture
description: Evalúa y define la arquitectura de la SPA React: Vite, TypeScript, organización por feature, estado y datos remotos, routing por rol, formularios, rendimiento y DX. Úsala al analizar o estructurar el frontend.
---

# Arquitectura frontend (React)

## Estructura objetivo
```
frontend/src/
  app/        router, providers (PrimeReact, QueryClient), layout por rol
  features/   agenda-publica, agenda, pacientes, ficha, recetas, pagos, staff
              └ components/ hooks/ api.ts types.ts
  shared/     UI reutilizable, utils (fechas, dinero)
  api/        cliente HTTP (fetch wrapper), tipos generados de OpenAPI
```

## Decisiones a evaluar (con evidencia)
| Tema | Recomendación base | Alternativas |
|---|---|---|
| Build | Vite | — |
| Lenguaje | TypeScript `strict` | JS (no recomendado) |
| Datos remotos | TanStack Query | SWR, fetch manual |
| Estado global | Mínimo: contexto de sesión/rol | Zustand si crece |
| Routing | React Router con guards por rol | — |
| Formularios | React Hook Form + Zod | Formik |
| UI | PrimeReact; React Bits solo efectos | — |
| Calendario | `@fullcalendar/react` | — |
| Fechas | date-fns/Day.js con zona de la sucursal | — |

## Rendimiento
Lazy loading por ruta, code splitting del backoffice vs agenda pública, importar solo componentes PrimeReact usados, medir bundle (`vite build --mode analyze`/rollup-plugin-visualizer).

## Reglas
- Sin `fetch` en componentes; todo por `src/api/`.
- Sin reglas de negocio críticas (precios, disponibilidad, permisos) en el cliente: solo presentación.
- Solo `VITE_*` públicas.

## Salida (análisis)
Tabla decisión/estado actual/recomendación/riesgo/esfuerzo con evidencia.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
