---
name: frontend-testing
description: Diseña y escribe pruebas del frontend con Vitest + Testing Library (y MSW para la API), priorizando flujos de usuario. Úsala al crear o modificar componentes y pantallas.
---

# Pruebas frontend

## Stack
Vitest + @testing-library/react + @testing-library/user-event + MSW (mock de la API) + vitest-axe. E2E opcional: Playwright para el flujo de reserva pública.

## Qué probar (prioridad)
1. Flujo de reserva pública: elegir sucursal → ver horarios → seleccionar → completar datos → confirmación; error 409 (horario tomado) muestra mensaje y refresca.
2. Guards por rol: rutas del backoffice inaccesibles sin sesión o con rol incorrecto.
3. Formularios: validación, mensajes de error, envío deshabilitado mientras carga.
4. Hooks de datos: estados loading/error/empty.
5. Formateo de fechas (zona de la sucursal) y dinero.

## Convenciones
- Consultas por rol/label (`getByRole('button', { name: /reservar/i })`), no por clases o test-ids salvo necesidad.
- Sin snapshots grandes.
- Handlers MSW tipados con los contratos de `src/api/`.
- Cada prueba independiente; resetear handlers en `afterEach`.

## Ejecución
`npm test` · `npx vitest run src/features/agenda-publica` · cobertura: `npx vitest run --coverage`.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
