---
name: technical-research
description: Investiga documentación, librerías y límites de servicios con fuentes fechadas y separa hechos de hipótesis.
---

# technical-research

## Cuándo usarla
Antes de una decisión de arquitectura, de elegir una librería o de configurar un servicio externo.

## Procedimiento
1. Reformula la pregunta en una frase verificable y define el time-box.
2. Busca primero la documentación oficial vigente; luego issues/changelogs del repositorio oficial; al final fuentes de terceros.
3. Registra cada afirmación con enlace y fecha de consulta. Marca **[H]** lo verificado y **[Hip]** lo inferido.
4. Si comparas opciones: tabla `opción | madurez | licencia | mantenimiento | compatibilidad con el stack | riesgo | fuente`.
5. Cierra con recomendación y qué falta verificar.

## Límites
No modifica el producto. Prototipos solo desechables y fuera de `frontend/` y `backend/`. Nunca usa ni pide credenciales.
