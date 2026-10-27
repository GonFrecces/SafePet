---
name: acceptance-criteria
description: Escribe y revisa criterios de aceptación verificables (dado/cuando/entonces) para issues de SafePet, detectando ambigüedad y casos borde. Úsala al crear issues o al revisar si una tarea está realmente terminada.
---

# Criterios de aceptación

## Formato
```
- Dado <estado inicial>, cuando <acción>, entonces <resultado observable>.
```
Cada criterio debe ser **observable**, **binario** (pasa/no pasa) y **ligado a una prueba**.

## Señales de ambigüedad (reescribir)
"rápido", "correctamente", "amigable", "seguro", "etc.", "manejar errores", "soportar móvil" sin umbral ni caso concreto.

## Casos borde a considerar en SafePet
- Otro tenant intenta leer/modificar (debe recibir 404 y no cambiar nada).
- Rol incorrecto (403).
- Horario ya tomado / reserva simultánea (409).
- Zona horaria de la sucursal y cambio de horario de verano.
- Cita en el pasado, fuera de disponibilidad, sucursal inactiva.
- Webhook duplicado o con firma inválida.
- Ancho móvil 360 px.

## Ejemplo
Malo: "La agenda pública funciona bien en móvil."
Bueno:
- Dado una sucursal con disponibilidad, cuando un cliente sin sesión abre la agenda en 360 px de ancho, entonces ve los horarios del día sin scroll horizontal.
- Dado un horario reservado por otro cliente, cuando intento reservarlo, entonces recibo "Horario no disponible" y la lista se actualiza.

## Checklist de "terminado"
- [ ] Todos los criterios tienen prueba y pasan. - [ ] Pruebas afectadas ejecutadas y reportadas. - [ ] Exclusiones respetadas. - [ ] Rollback documentado.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
