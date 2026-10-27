---
name: dependency-audit
description: Inventaria dependencias .NET y npm con versión actual, última, estado de soporte y vulnerabilidades conocidas, sin modificar nada. Úsala en auditorías y antes de planificar actualizaciones.
---

# Auditoría de dependencias

## Comandos (solo lectura)
.NET:
```bash
dotnet list package --include-transitive
dotnet list package --outdated
dotnet list package --vulnerable --include-transitive
```
npm:
```bash
npm ls --depth=0
npm outdated
npm audit --omit=dev --json
```
Si requieren `restore`/`npm ci` y eso escribiría en el repo, decláralo y usa los manifiestos (`*.csproj`, `package.json`, lockfile).

## Clasificación
| Estado | Criterio |
|---|---|
| OK | Versión soportada, sin CVE |
| Desactualizada menor | Hay minor/patch nuevos |
| Desactualizada mayor | Cambio mayor disponible (riesgo de breaking) |
| Sin soporte | Runtime/framework fuera de soporte (ej. .NET fuera de LTS/STS vigente) |
| Vulnerable | CVE/GHSA conocido; anota severidad y si es explotable en este uso |

## Salida
Tabla `ecosistema | paquete | actual | última | tipo (directa/transitiva) | estado | nota`.
Luego: plan de actualización **una dependencia mayor por PR**, ordenado por riesgo de seguridad y por dependencias entre paquetes; nunca actualización masiva.

## Reglas
- No ejecutar `npm audit fix`, `npm update`, `dotnet add package`.
- Citar fuente de soporte (página oficial de ciclo de vida) con fecha de consulta.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
