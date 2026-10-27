---
name: test-driven-development
description: Aplica TDD (rojo → verde → refactor) en .NET y React para implementar comportamiento nuevo o corregir bugs con una prueba que falla primero. Úsala al implementar features o fixes.
---

# Test-Driven Development

## Ciclo
1. **Rojo.** Escribe una prueba que exprese un criterio de aceptación. Ejecútala y **confirma que falla por la razón esperada** (no por compilación o setup).
2. **Verde.** Implementa lo mínimo para que pase. Ejecuta la prueba y las relacionadas.
3. **Refactor.** Mejora nombres/estructura sin cambiar comportamiento; las pruebas siguen en verde.
4. Commit pequeño. Repite con el siguiente criterio o caso borde.

## Para bugs
Primero una prueba que reproduce el bug (falla), luego el fix. La prueba queda como regresión.

## Comandos
- .NET: `dotnet test --filter "FullyQualifiedName~<Clase>"` en el ciclo; suite afectada al final.
- Front: `npx vitest <archivo>` (watch) en el ciclo; `npm test` al final.

## Buenas prácticas
- Una razón para fallar por prueba; nombre `Metodo_Escenario_Resultado`.
- Arrange/Act/Assert claros; builders para datos.
- Prueba comportamiento observable, no detalles internos.
- Incluye el caso multi-tenant cuando el código toca datos de clínica.

## Evidencia en el PR
Menciona la prueba que falló primero y el comando que la vio pasar.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
