# AGENTS.md — ticketflow-eureka-server

Service discovery (Eureka) de TicketFlow. Es el **primer servicio en levantar**: el resto se registra contra él.

## Servicio

| Campo | Valor |
|---|---|
| Rol | Service discovery (Eureka Server) |
| Eureka name | `eureka-server` |
| Puerto | 8761 |
| Depende de | — |
| Stack | Java 21, Spring Boot 4.1.1, Spring Cloud 2025.1.3 |

## Comandos

```powershell
.\mvnw.cmd test              # compilar y testear
.\mvnw.cmd spring-boot:run   # levantar Eureka en :8761
```

## Gotchas

- Debe estar arriba antes que el resto; los demás se registran vía `eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka`.
- El `README.md` está vacío: no lo uses como fuente.
- El Dockerfile va en minúscula (`dockerfile`), así lo referencian los compose.

## Convenciones

- Arquitectura Hexagonal + Clean + DDD cuando aplique; el dominio queda en Java puro.
- Código, identificadores y comentarios **en inglés**; documentación en español.

## Gobernanza (docs_ia)

- `docs_ia/contexto/` es la **única fuente autorizada** de requisitos, arquitectura, calidad y pruebas. No asumas lo que no esté ahí.
- Decisiones arquitectónicas: proponer en `docs_ia/decisiones/DA-xxx-*.md` (estado `PROPUESTA`) y **esperar aprobación humana** antes de implementar. No sobrescribir decisiones previas.
- Trazabilidad: registrar cada interacción relevante en `docs_ia/prompts/P-xxx-*.md`.
- No agregar/quitar/actualizar dependencias ni modificar archivos (salvo trazabilidad/propuestas) sin autorización humana explícita.

## Grafo (codebase-memory)

- Proyecto: `C-Users-Juan-Diego-Duque-Documents-Programacion-ticketflow-ticketflow-eureka-server`.
- Este repo es git → `detect_changes --base-branch main` funciona.
- Arquitectura entre servicios → grafo padre `C-Users-Juan-Diego-Duque-Documents-Programacion-ticketflow`.
- **Cambiar de rama NO refresca este grafo automáticamente**: tras `git checkout`, fuerza re-index con `index_repository --repo-path . --mode full`.

## No tocar

- `.mvn/`, `.idea/`, `target/` = build/IDE output.
