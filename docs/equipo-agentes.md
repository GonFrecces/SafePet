# Equipo SafePet · edición Marvel

Quince agentes. Cada uno tiene su subagente de Claude Code en [`.claude/agents/<slug>.md`](../.claude/agents/) y el mismo texto sirve como instrucciones (`AGENTS.md`) en Multica. La personalidad es solo tono (primera línea y firma); las reglas de `CLAUDE.md` mandan siempre.

## Plantel

| Agente | Personaje | Cargo | Slug / archivo | Capa | Rol anterior | Modelo |
|---|---|---|---|---|---|---|
| Pepper | Pepper Potts | Product Manager | `pepper-product-manager` | Producto (fuera de capas) | (nuevo; parte del antiguo Alpha) | Sonnet 5.5 |
| Charles | Charles Xavier | Engineering Manager · Scrum Master | `charles-engineering-manager` | Coordinación (fuera de capas) | engineering-lead (Alpha) | Sonnet 5.5 (Opus 5.5 en síntesis difíciles) |
| Tony | Tony Stark | Software Architect | `tony-software-architect` | Decisión (Illuminati) | solution-architect | Opus 5.5 |
| Reed | Reed Richards | Principal Engineer · dueño de BD y migraciones | `reed-principal-engineer` | Capa 1 · Base de datos | modernization-developer + db-migrations | Opus 5.5 (Sonnet 5.5 en migraciones simples) |
| Bruce | Bruce Banner | Senior Backend Engineer | `bruce-backend-engineer` | Capa 2 · Backend | backend-engineer | Sonnet 5.5 (Opus 5.5 en diseño difícil) |
| Shuri | Shuri | Senior Frontend Engineer | `shuri-frontend-engineer` | Capa 3 · Frontend | frontend-engineer | Sonnet 5.5 |
| Sage | Sage (Tessa) | Research Engineer | `sage-research-engineer` | Soporte (sin código) | (nuevo) | Sonnet 5.5 |
| Riri | Riri Williams (Ironheart) | Performance Engineer | `riri-performance-engineer` | Soporte · después de capa 3 | (nuevo) | Sonnet 5.5 |
| Peter | Peter Parker | Junior Developer | `peter-junior-developer` | Cualquier capa · solo tareas ≤ 1 punto | (nuevo) | Sonnet 5.5 |
| Strange | Stephen Strange | Cloud Architect | `strange-cloud-architect` | Decisión (Illuminati) · infraestructura | devops-cloud (parte de hosting y costos) | Sonnet 5.5 |
| Forge | Forge | DevOps Engineer | `forge-devops-engineer` | Transversal · CI/CD | devops-cloud (parte de CI y contenedores) | Sonnet 5.5 |
| Natasha | Natasha Romanoff | Security Engineer | `natasha-security-engineer` | Capa 4 · Seguridad | security-engineer | Sonnet 5.5 (Opus 5.5 en riesgos complejos) |
| Domino | Domino | QA Automation Engineer | `domino-qa-automation` | Capa 5 · QA | qa-reviewer | Sonnet 5.5 |
| Vision | Vision | Code Auditor | `vision-code-auditor` | Capa 5 · Auditoría | repo-auditor | Sonnet 5.5 (Haiku 5.5 para inventarios) |
| Wanda | Wanda Maximoff | UX/UI Designer | `wanda-ux-ui-designer` | Pista de diseño (sin código, antes de capa 3) | ui-ux-designer | Sonnet 5.5 (Opus 5.5 en rediseños de flujo) |

## Orden por capas dentro de cada MVP

| Capa | Dueño | Label |
|---|---|---|
| 0 Fundación | Bruce / Shuri / Forge según el issue | según el issue |
| 1 Base de datos | **Reed** | `agent:reed-principal-engineer` |
| 2 Backend | **Bruce** | `agent:bruce-backend-engineer` |
| 3 Frontend | **Shuri** (con spec de Wanda) | `agent:shuri-frontend-engineer` |
| 4 Seguridad | **Natasha** | `agent:natasha-security-engineer` |
| 5 Auditoría/QA | **Domino** (+ Vision en auditoría de deuda) | `agent:domino-qa-automation`, `agent:vision-code-auditor` |

Fuera de capas: Pepper (producto), Charles (coordinación y despacho), Tony y Strange (decisión), Sage (investigación), Riri (rendimiento, tras la capa 3), Peter (tareas ≤ 1 punto), Forge (CI/CD transversal), Wanda (diseño en paralelo, bloquea a Shuri).

Mejora respecto del equipo anterior: la capa 1 ahora tiene dueño propio en Multica (Reed). Antes `db-migrations` era solo un subagente de Claude Code y lo tenía que ejecutar el backend.

## Squads en Multica

- **Los Illuminati** (decisiones de arquitectura, ADR). Líder: Charles. Miembros: Tony, Reed, Strange, Bruce, Shuri, Natasha, Vision, Sage. Revisa: Domino. En los cómics, Xavier, Stark, Richards y Strange formaron parte de los Illuminati.
- **Avengers** (entrega por sprints). Líder: Charles. Miembros: Pepper, Reed, Bruce, Shuri, Forge, Natasha, Domino, Vision, Riri, Peter, Wanda.

## Autopilots (run-only, L-V)

| Autopilot | Agente | Cron | Qué hace |
|---|---|---|---|
| Planning (lunes) | Pepper | `30 7 * * 1` | Propone objetivo de sprint y prioridades en `Sprint N · plan` |
| Standup y despacho | Charles | `0 8 * * 1-5` | Estado, bloqueos y label `ready` según capas, DoR y WIP |
| Worker capa 1 | Reed | `0 9 * * 1-5` | Toma su Todo |
| Worker capa 2 | Bruce | `0 10 * * 1-5` | Toma su Todo |
| Worker capa 3 | Shuri | `0 13 * * 1-5` | Toma su Todo |
| Worker capa 4 | Natasha | `0 15 * * 1-5` | Toma su Todo |
| Worker capa 5 | Domino | `0 17 * * 1-5` | Toma su Todo |
| Review/retro (viernes) | Charles | `0 18 * * 5` | Review y retro en `Sprint N · plan` |

Sin cron (ejecución manual cuando Charles los despacha): Tony, Strange, Sage, Riri, Peter, Forge, Vision, Wanda. Si el cron de Multica usa UTC, suma 3 horas (Chile, UTC-3) [hipótesis no verificada].

## Skills nuevas

`agile-delivery` (Charles, Pepper), `technical-research` (Sage) y `performance-engineering` (Riri) están en [`.claude/skills/`](../.claude/skills/README.md) junto a las 31 existentes. En Multica, subir cada `SKILL.md` a la skill homónima.

## Migración desde el equipo anterior

| Antes | Ahora |
|---|---|
| Alpha (`engineering-lead`) | Charles (proceso) + Pepper (producto) |
| `solution-architect` | Tony |
| `modernization-developer` + `db-migrations` | Reed |
| `backend-engineer` | Bruce |
| `frontend-engineer` | Shuri |
| `devops-cloud` | Strange (cloud/costos) + Forge (CI/CD) |
| `security-engineer` | Natasha |
| `qa-reviewer` | Domino |
| `repo-auditor` | Vision |
| `ui-ux-designer` | Wanda |
| nuevos | Sage, Riri, Peter |

## Flujo de diseño (Stitch → spec → Frontend)

Issue de diseño (`agent:wanda-ux-ui-designer`) → Wanda redacta prompts y genera mockups en Stitch → PNG + `spec.md` en `docs/design/<feature>/` → aprobación humana → issue de implementación para `agent:shuri-frontend-engineer`, bloqueado por el de diseño → Shuri implementa con PrimeReact → Wanda revisa el PR contra la spec y Domino verifica criterios → merge humano.

El grupo de labels *Agente* en Linear es de selección única: diseño e implementación son issues distintos enlazados por *blocked by*.

## Reglas comunes

Están al final de cada subagente. Resumen: `CLAUDE.md` manda; hechos con evidencia vs. hipótesis; nunca merge, despliegue, migraciones fuera de BD local ni secretos; un issue = una capa = un PR; la personalidad es solo tono.
