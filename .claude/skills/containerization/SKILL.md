---
name: containerization
description: Crea y revisa Dockerfiles multi-stage para ASP.NET Core y docker-compose para desarrollo local con PostgreSQL. Úsala al contenerizar el backend o preparar el entorno local.
---

# Contenedores

## Dockerfile backend (multi-stage)
```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:<ver> AS build
WORKDIR /src
COPY backend/*.sln backend/
COPY backend/src/*/*.csproj ...   # copiar csproj primero para cachear restore
RUN dotnet restore backend/SafePet.sln
COPY backend/ backend/
RUN dotnet publish backend/src/SafePet.Api -c Release -o /app --no-restore

FROM mcr.microsoft.com/dotnet/aspnet:<ver> AS runtime
WORKDIR /app
COPY --from=build /app .
USER app
ENV ASPNETCORE_URLS=http://+:8080
EXPOSE 8080
HEALTHCHECK CMD wget -qO- http://localhost:8080/health || exit 1
ENTRYPOINT ["dotnet", "SafePet.Api.dll"]
```
Ajustar `<ver>` a la versión del proyecto (fijada, no `latest`).

## Reglas
- Usuario no root; imagen runtime mínima (aspnet o chiseled).
- `.dockerignore`: `bin/ obj/ node_modules/ .git .env* **/appsettings.*.local.json`.
- Configuración solo por variables de entorno; ningún secreto en la imagen ni en `ARG`.
- Puerto desde `PORT` si el proveedor lo exige (Render/Fly).

## docker-compose (local)
`postgres:<ver>` con volumen, healthcheck y credenciales de desarrollo en `.env` (ignorado) con `.env.example` de plantilla.

## Verificación
`docker build .` local, `docker run` y `GET /health` → 200. Tamaño de imagen reportado.

---
Reglas transversales: `CLAUDE.md` del repositorio (sin secretos, sin merge ni despliegue, hechos vs. hipótesis).
