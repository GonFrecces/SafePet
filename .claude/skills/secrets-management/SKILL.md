---
name: secrets-management
description: Detecta y previene secretos en el repositorio y define cómo gestionarlos (variables de entorno, user-secrets, gestores del proveedor, rotación). Úsala en revisiones, al configurar entornos o si aparece un secreto.
---

# Gestión de secretos

## Qué es secreto
Cadenas de conexión con contraseña, `sk_*`/`rk_*`/`whsec_*` de Stripe, claves de firma JWT, tokens de API (email/SMS), credenciales Neon/Supabase (service role), certificados y claves privadas.
`pk_*` de Stripe y URLs públicas no son secretos, pero igual van por variable de entorno.

## Dónde viven
| Entorno | Mecanismo |
|---|---|
| Local backend | `dotnet user-secrets` |
| Local frontend | `.env.local` (ignorado) con solo `VITE_*` públicas |
| CI | GitHub Actions secrets / Environments |
| Hosting | Gestor de variables del proveedor |
| Repo | Solo plantillas: `.env.example`, `appsettings.Example.json` con valores ficticios |

## Detección
- `git grep -nE "(sk|rk)_(live|test)_|whsec_|Password=|-----BEGIN .*PRIVATE KEY-----"` (reportar archivo:línea, nunca el valor).
- Opcional: gitleaks / GitHub secret scanning en CI.
- Verificar `.gitignore` y `.claude/settings.json` (deny de lectura de `.env`).

## Si aparece un secreto commiteado
1. Detener el trabajo. 2. Avisar al responsable. 3. **Rotar** el secreto en el proveedor. 4. Eliminarlo del código y moverlo a variable. 5. Reescribir historial no basta y nunca sustituye la rotación.

## Reglas
Nunca pedir, pegar ni reproducir secretos en issues, PRs, logs o prompts.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
