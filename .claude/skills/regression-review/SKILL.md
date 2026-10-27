---
name: regression-review
description: Revisa un cambio o propuesta buscando regresiones: comportamiento existente roto, contratos API/BD incompatibles, pruebas debilitadas y efectos en otros módulos. Úsala antes de aprobar un PR o una síntesis de decisión.
---

# Revisión de regresiones

## Procedimiento
1. **Alcance real.** `git diff --stat origin/master...HEAD`; compara con el alcance declarado del issue. Señala archivos fuera de alcance.
2. **Contratos.**
   - API: campos renombrados/eliminados, cambios de tipo o semántica, códigos de estado nuevos. ¿Se actualizó el frontend?
   - BD: migraciones destructivas sin expand→contract; `Down` ausente; datos existentes que violan nuevos `NOT NULL`/únicos.
3. **Pruebas.** ¿Se borraron, omitieron o relajaron aserciones? ¿Cambió un snapshot sin motivo? ¿Las nuevas pruebas fallarían sin el cambio?
4. **Comportamiento.** Recorre flujos críticos afectados (reserva pública, abrir/cerrar cita, cobro, recordatorio) y pregunta qué pasa con datos existentes.
5. **Multi-tenant.** ¿Nuevas consultas respetan el filtro? ¿Prueba de aislamiento incluida?
6. **Errores preexistentes.** Compara fallos con el registro del Auditor para no atribuirlos al PR (ni ocultarlos).
7. **Ejecuta** las pruebas afectadas (CLAUDE.md §7) y reporta.

## Para propuestas (no código)
Busca contradicciones entre informes, supuestos sin evidencia, decisiones que rompen reglas de CLAUDE.md, riesgos omitidos (rollback, compatibilidad, costo).

## Salida
Objeciones bloqueantes / no bloqueantes con evidencia, comandos ejecutados, veredicto `listo` / `requiere cambios`.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
