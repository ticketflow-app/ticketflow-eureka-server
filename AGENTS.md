# AGENTS.md — ticketflow-eureka-server

Service discovery (Eureka Server) de TicketFlow. Es el **primer servicio en levantar**: el resto se registra contra él.

> Este archivo orienta; **la fuente de verdad del proyecto es `docs_ia/contexto`**. El diseño, los requisitos y la arquitectura no se describen aquí.

## Estado

| Campo | Valor |
|---|---|
| Rol | Service discovery (Eureka Server) |
| Eureka name | `eureka-server` |
| Puerto | 8761 — coincide con la especificación |
| Depende de | — |
| Stack (estándar del proyecto) | **Java 25 LTS**, **Spring Boot 4.1.1**, **Spring Cloud 2025.1.3**, Spring Cloud Netflix Eureka Server **5.0.x** — hoy en **Java 21** ⚠️ |
| Prefijo de rutas | **Exento**: solo expone la API del framework (`/`, `/eureka/**`) y `/actuator/**` |
| Contrato HTTP | `docs_ia/contexto/contrato_api.md` |

## Contrato y endpoints

Detalle completo (endpoints del registro, configuración y desviaciones): `docs_ia/contexto/contrato_api.md`.
Descripción de la arquitectura: `docs_ia/contexto/arquitectura/arquitectura.md`.

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/` | Panel web de Eureka |
| GET | `/eureka/apps` | Instancias registradas (JSON o XML) |
| GET | `/eureka/apps/{app}` | Instancias de una aplicación |
| POST | `/eureka/apps/{app}` | Registro de una instancia |
| PUT | `/eureka/apps/{app}/{id}` | Latido / renovación de lease |
| GET | `/eureka/apps/{app}/{id}` | Latido de una instancia concreta |
| DELETE | `/eureka/apps/{app}/{id}` | Baja de la instancia |
| POST | `/eureka/apps/{app}/{id}/status?value=OUT_OF_SERVICE` | Marcar instancia como no disponible |

Clientes registrados: `api-gateway` (8080), `ticketflow-user-service` (8081),
`ticketflow-inventory-service` (8082).

Desviaciones más relevantes frente a la especificación de plataforma (`docs/Microservicios.docx`,
sección 3):

- **Sin Basic Auth**: la spec dice «protegido con Basic Auth»; el panel y el registro están abiertos (solo aceptable en local).
- **Autoprotección desactivada globalmente** (`enable-self-preservation=false`): debería activarse por perfil de entorno.
- **Registro solo en memoria**: un reinicio lo vacía.
- **Java 21** en lugar del estándar del proyecto (**25 LTS**).
- **Exento de la norma `/api/`**: sus endpoints son del framework; prefijarlos rompería el descubrimiento de servicios (ver `docs_ia/contexto/contrato_api.md` §5).

## Docs

- `docs_ia/contexto/contexto_negocio.md` — contexto de negocio.
- `docs_ia/contexto/contrato_api.md` — **contrato HTTP vigente**: endpoints del registro, configuración del servidor y desviaciones frente a la especificación de plataforma.
- `docs_ia/contexto/arquitectura/` — `arquitectura.md` (configuración, ciclo de vida de clientes, endpoints), `desiciones_arquitectonicas.md`.
- `docs_ia/contexto/requisitos/` — `casos_uso.md`, `epicas_feactures.md`, `historias_usuarios.md`.
- `docs_ia/contexto/calidad/` — `atributos_calidad.md`, `escenarios_calidad.md`.
- `docs_ia/contexto/pruebas/` — `estrategia_pruebas.md`, `pruebas_cargas.md`.
- `docs_ia/decisiones/` — decisiones arquitectónicas (`DA-xxx`).
- `docs_ia/prompts/` — trazabilidad de uso de IA (`P-xxx`).

## Comandos

```powershell
.\mvnw.cmd test              # compilar y testear
.\mvnw.cmd spring-boot:run   # levantar Eureka en :8761
```

## Gobernanza (para OpenCode)

> Adaptación para **OpenCode** de las reglas de `.clinerules` (ver el archivo completo en los repos que lo tienen: `ticketflow-inventory-service`, `ticketflow-frontend`). Aplica a todo el trabajo en este repo; su fuente de verdad es `docs_ia/contexto`.

### 1. Fuente de contexto

- `docs_ia/contexto` es la **única fuente autorizada** para: requisitos, atributos de calidad, arquitectura, diseño, estrategia de pruebas y restricciones del proyecto.
- No asumir información que no esté definida en `docs_ia/contexto`.
- Antes de una tarea relevante, revisar los archivos de `docs_ia/contexto` necesarios para esa tarea.
- Si falta información, distinguir entre el hecho conocido, la inferencia y la propuesta; si la tarea entra en conflicto con el contexto, detener el cambio afectado y pedir aclaración.

### 2. Decisiones arquitectónicas

- No tomar decisiones arquitectónicas importantes de forma autónoma.
- Cuando haga falta decidir, analizar alternativas y proponer una recomendación.
- Crear la propuesta en `docs_ia/decisiones/` en Markdown, usando el siguiente identificador `DA-xxx` disponible.
- Esperar aprobación humana antes de implementar la decisión propuesta.
- Una decisión aprobada queda documentada en `docs_ia/decisiones/`.
- No sobrescribir decisiones anteriores; si una decisión cambia, crear una nueva que explique el cambio.
- Cada decisión debe incluir: contexto, problema, alternativas consideradas, recomendación, ventajas, desventajas, impacto y estado.
- Estado `PROPUESTA` hasta recibir aprobación humana explícita; solo entonces `APROBADA` y aplicable.

### 3. Código

- Puede leer y modificar el código cuando la tarea haya sido aprobada.
- No hacer cambios importantes basados en suposiciones.
- Antes de cambios arquitectónicos, explicar brevemente qué se modificará y por qué.
- Mantener coherencia con las decisiones aprobadas.
- Nomenclatura **en inglés**: variables, métodos, clases, interfaces, paquetes, endpoints, DTOs, entidades, tablas, columnas y archivos de código.
- Comentarios y documentación técnica dentro del código, **en inglés**.
- No usar nombres en español en el código, salvo nombres externos obligados por una API, librería o requisito existente.
- Nomenclatura consistente y descriptiva según el lenguaje y framework.

### 4. Documentación

- Toda decisión arquitectónica o técnica importante se documenta en Markdown.
- Identificadores consecutivos: `DA-001`, `DA-002`, `DA-003`, ...
- No sobrescribir decisiones anteriores; si una decisión cambia, crear una nueva que explique el cambio.
- Cada decisión debe incluir: contexto, problema, alternativas consideradas, recomendación, ventajas, desventajas, impacto y estado.
- Estado `PROPUESTA` hasta recibir aprobación humana explícita; solo entonces `APROBADA` y aplicable.
- Documentación de proyecto en **español**; código y comentarios técnicos en **inglés**.

### 5. Pruebas

- No modificar la estrategia de pruebas definida en `docs_ia/contexto` sin proponerlo antes.
- Cuando corresponda, crear o modificar pruebas junto con el código.
- No introducir nuevas herramientas o frameworks de pruebas sin analizar antes su necesidad y compatibilidad.
- Las pruebas deben verificar comportamientos concretos, no limitarse a confirmar que las clases existen.

### 6. Gestión de dependencias

- Antes de agregar, eliminar, actualizar o reemplazar una dependencia, revisar los archivos relevantes de `docs_ia/contexto`.
- Revisar siempre el manifiesto de dependencias actual (`pom.xml`) antes de proponer cambios.
- Clasificar las dependencias como: indispensables, recomendadas, opcionales o no justificadas actualmente.
- Para cada dependencia propuesta indicar: propósito, funcionalidad, relación con la arquitectura, requisito que satisface, alternativas, impacto técnico y compatibilidad con las versiones actuales.
- Verificar compatibilidad con: versión de Java, versión de Spring Boot, versión de Maven y arquitectura definida.
- No agregar dependencias por conveniencia, costumbre o sugerencia automática de una herramienta.
- Principio de mínima dependencia: solo las justificadas por requisitos y arquitectura aprobada.
- **No agregar, eliminar, actualizar ni modificar dependencias sin autorización humana explícita.**
- Antes de modificar el manifiesto, presentar análisis y propuesta; tras la autorización, aplicar solo lo aprobado.
- Toda modificación importante de dependencias se documenta en `docs_ia/decisiones`.

### 7. Uso de herramientas

- Puede leer archivos; crear y modificar cuando la tarea esté aprobada; ejecutar comandos necesarios para la tarea aprobada.
- No ejecutar comandos que modifiquen el proyecto sin autorización cuando los cambios no estén aprobados.
- Pedir aprobación antes de comandos destructivos o peligrosos y antes de instalar paquetes, dependencias, herramientas o software.
- No eliminar archivos, dependencias, configuraciones o código sin autorización explícita.
- Si una herramienta propone cambios automáticos, analizar primero y pedir autorización antes de aplicarlos.

### 8. Autorización de cambios

- Toda tarea que modifique código, configuración, arquitectura, dependencias o infraestructura requiere autorización humana si no estaba previamente aprobada.
- Analizar primero, modificar después.
- Una autorización específica no autoriza cambios adicionales no relacionados.
- No interpretar expresiones ambiguas como autorización para cambios importantes.
- Si la tarea requiere una decisión no definida en `docs_ia/contexto`, detener la implementación y presentar la decisión.
- El permiso de lectura no implica permiso de escritura: cada cambio debe caer dentro del alcance autorizado.

### 9. Comunicación

- Explicar las decisiones de forma breve y técnica.
- Si falta información en `docs_ia/contexto`, indicarlo explícitamente.
- Diferenciar siempre entre: información definida, suposición, propuesta y decisión aprobada.
- Presentar las alternativas técnicamente válidas antes de elegir una.
- No presentar una propuesta como si fuera una decisión aprobada.
- Informar qué archivos se modificarán antes de cambios importantes.
- Tras cambios autorizados, informar: archivos modificados, cambios realizados, comandos ejecutados, pruebas realizadas, resultado y pendientes.

### 10. Trazabilidad del uso de IA

- Toda interacción relevante debe quedar registrada, incluso si es solo análisis sin cambios de código.
- Se registra desde que se ejecuta el prompt y no requiere autorización.
- Ubicación: `docs_ia/prompts`, un archivo Markdown independiente por interacción, con identificadores consecutivos `P-001`, `P-002`, ...
- Contenido mínimo: identificador, fecha, objetivo, prompt completo, archivos de contexto consultados, archivos analizados, resumen de la respuesta, propuestas, decisiones pendientes y estado.
- Estados permitidos: `ANALISIS` | `PROPUESTA` | `APROBADO` | `RECHAZADO` | `IMPLEMENTADO`.
- Si hay propuesta arquitectónica o técnica, crear también el documento correspondiente en `docs_ia/decisiones`.
- El registro de trazabilidad y la propuesta documental **no** autorizan a modificar código, configuración, dependencias o infraestructura.
- Ningún archivo (salvo trazabilidad y propuestas) se modifica sin autorización humana explícita.
- Si solo se pide análisis, ejecutarlo y documentarlo sin tocar código.
- Si luego se autoriza, actualizar el registro con autorización, cambios, archivos, comandos, pruebas, resultado y estado final.
- Nunca sobrescribir registros históricos; si algo cambia, crear un nuevo registro relacionado.
- La trazabilidad debe permitir reconstruir: Contexto → Prompt → Respuesta IA → Propuesta → Autorización humana → Implementación → Pruebas → Resultado.

## Grafo (codebase-memory)

- Proyecto de grafo: `C-Users-Juan-Diego-Duque-Documents-Programacion-ticketflow-ticketflow-eureka-server`.
- Este repo es git → `detect_changes --base-branch main` funciona contra este proyecto.
- Arquitectura entre servicios → grafo padre `C-Users-Juan-Diego-Duque-Documents-Programacion-ticketflow` (foto del estado actual de los 5 checkouts).
- **Cambiar de rama NO refresca este grafo automáticamente** (el watcher sigue la raíz, que no es git). Tras `git checkout` o al editar, fuerza re-index:
  ```powershell
  powershell -File ..\sync-graph.ps1 -Service ticketflow-eureka-server   # desde la raíz
  ```
  o, desde esta carpeta: `index_repository --repo-path . --mode full`.

## Gotchas

- Debe estar arriba **antes que el resto**: los demás se registran vía `eureka.client.serviceUrl.defaultZone=http://localhost:8761/eureka`.
- Es el único que **no** se registra: su configuración trae `eureka.client.register-with-eureka=false` y `fetch-registry=false`.
- Self-preservation desactivado en dev (`enable-self-preservation=false`) con `eviction-interval-timer-in-ms: 5000`: los servicios caídos se expulsan rápido en lugar de quedar "protegidos".
- El dashboard se sirve desde el propio jar en `http://localhost:8761`: no hay archivos HTML en el repo que lo implementen.
- El `README.md` está vacío: no lo uses como fuente.
- El Dockerfile va en minúscula (`dockerfile`), así lo referencian los compose.
- **Sin Basic Auth**: la especificación de plataforma dice «protegido con Basic Auth». El panel (`/`) y el registro (`/eureka/apps`) están abiertos → solo aceptable en local.
- **El registro no es persistente**: vive en memoria. Un reinicio lo vacía y los clientes se re-registran solos, pero hay una ventana sin resolución de nombres.
- **No Prometheus**: el starter de Eureka no expone `/actuator/prometheus` (la especificación lo pide para AWS).

## No tocar

- `.mvn/`, `.idea/`, `target/` = build/IDE output.
