---
name: repository-audit
description: Audita un repositorio en modo solo lectura y produce un informe con evidencia: estructura, stack y versiones, comandos, pruebas, configuración y riesgos. Úsala antes de cualquier decisión de arquitectura o modernización.
---

# Auditoría de repositorio

## Reglas
Solo lectura. Prohibido: editar, instalar, actualizar, `git clean/reset`, `npm audit fix`, migraciones. Al terminar, `git status` debe estar limpio.

## Procedimiento
1. **Identidad.** `git log --oneline -20`, ramas, último commit, tamaño (`git count-objects -vH`).
2. **Estructura.** `find . -maxdepth 3 -type d -not -path './.git*'`; localiza `*.sln`, `*.csproj`, `package.json`, `Dockerfile`, `.github/workflows`, migraciones.
3. **Stack.** Target frameworks (`<TargetFramework>`), versión de Node (`engines`, `.nvmrc`), frameworks y librerías principales.
4. **Comandos.** Compara los documentados (README, CLAUDE.md) con los reales (`scripts` de package.json, proyectos .NET). Ejecuta build/test solo si no escriben en el árbol versionado; registra salida resumida.
5. **Configuración.** `appsettings*.json`, `.env.example`, variables requeridas. Indicios de secretos: reporta archivo:línea, nunca el valor.
6. **Datos.** DbContext, migraciones, seeds, estrategia multi-tenant actual.
7. **Calidad.** Linters, formato, nullable, warnings como error, CI.
8. Delegar inventario detallado a `dependency-audit` y cobertura a `test-coverage-analysis`.

## Plantilla de informe
```
## Resumen ejecutivo (5 líneas)
## Estructura
## Stack y versiones
## Comandos (documentado vs real, con salida)
## Pruebas y errores preexistentes
## Configuración y secretos (sin valores)
## Riesgos (HECHO / HIPÓTESIS, severidad)
## Tareas sugeridas (prioridad, dependencia)
## Verificación: git status limpio
```

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
