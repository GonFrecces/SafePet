---
name: test-coverage-analysis
description: Analiza qué partes del código tienen pruebas y cuáles no, por capa y por riesgo, y propone dónde agregar pruebas primero. Úsala en auditorías o antes de refactorizar.
---

# Análisis de cobertura de pruebas

## Procedimiento
1. **Inventario de suites.** Proyectos `*.Tests`, carpetas `tests/`, `*.test.ts(x)`, `*.spec.*`. Framework (xUnit/NUnit, Vitest/Jest, Playwright).
2. **Ejecución (si es posible sin escribir en el repo).**
   - .NET: `dotnet test --collect:"XPlat Code Coverage" --results-directory /tmp/cov`
   - Front: `npx vitest run --coverage --coverage.reportsDirectory=/tmp/cov`
   Registra pasan/fallan/omitidas y errores preexistentes.
3. **Mapa por capa.** `capa/módulo | tiene pruebas | tipo | cobertura aprox | riesgo`.
4. **Priorización por riesgo, no por porcentaje.** Primero: aislamiento multi-tenant, autorización por rol, reserva de horas (concurrencia), pagos/webhooks, cálculo de montos, migraciones.
5. **Pruebas omitidas.** Lista de `[Skip]`, `.skip`, `xit` con motivo.

## Salida
- Tabla de cobertura por capa.
- Lista de huecos críticos con la prueba sugerida (nombre + nivel unitario/integración/e2e).
- Registro de fallos preexistentes (para que QA no los confunda con regresiones).

## Reglas
No borrar ni desactivar pruebas. Una cobertura alta no prueba calidad: verifica que haya aserciones significativas.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
