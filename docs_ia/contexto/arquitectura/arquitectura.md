# Arquitectura — ticketflow-eureka-server

> **Alcance de este documento:** describe **exclusivamente la arquitectura del servidor Eureka**.
> La arquitectura de cada servicio que se registra contra él está en su propio repositorio; aquí
> solo se nombran los clientes registrados porque el ciclo de vida del registro es responsabilidad
> de este servicio. No se describe el interior de ningún otro microservicio.

## 1. Propósito

Servidor **Eureka** de Spring Cloud Netflix: es el registro de servicios de la plataforma. Los
servicios cliente (gateway, user, inventory) se registran contra él y resuelven los destinos
`lb://<nombre-de-aplicación>` a través de él.

- Contrato HTTP del registro: `docs_ia/contexto/contrato_api.md`.
- Especificación de la plataforma: `docs/Microservicios.docx` (sección 3) y `docs/Arquitectura.docx`
  (secciones 2 y 6), en la raíz del workspace.

## 2. Identidad y despliegue

| Campo | Valor |
|---|---|
| Nombre de aplicación | `eureka-server` |
| Puerto | `8761` (`server.port`) |
| Prefijo de rutas | **Exento**: solo expone la API del framework (`/`, `/eureka/**`) y `/actuator/**`. Ver `docs_ia/contexto/contrato_api.md` §5. |
| Base de datos | ninguna — el registro se mantiene **en memoria** |
| Dependencia | ninguna (es el primer servicio en levantar) |
| Starter | `spring-cloud-starter-netflix-eureka-server` |
| Docker | imagen en el compose del gateway, `dockerfile` en minúscula |

### 2.1 Estándar de versiones

Versiones **obligatorias y uniformes para todo el proyecto**. Este repositorio solo declara las
tecnologías que **usa**.

| Tecnología | Versión estándar | Versión actual | Estado |
|---|---|---|---|
| Java | **25 (LTS)** | 21 | ⚠️ Pendiente de migración |
| Spring Boot | **4.1.1** | 4.1.1 | ✔ Al día |
| Spring Cloud | **2025.1.3** | 2025.1.3 | ✔ Al día |
| Spring Cloud Netflix Eureka Server | **5.0.x** | 5.0.x (BOM) | ✔ Al día |

> Este servicio **no** usa PostgreSQL, Redis, Kafka, jjwt ni Flyway: no aparecen en su stack.

## 3. Estilo arquitectónico

Servicio de **infraestructura transversal**, sin dominio de negocio. No aplica arquitectura
hexagonal: expone directamente el contrato HTTP que define Spring Cloud Netflix Eureka Server
(registro, latido, baja, consulta y panel web).

## 4. Configuración actual (`src/main/resources/application.yml`)

| Clave | Valor | Significado |
|---|---|---|
| `server.port` | `8761` | Puerto del registro y del panel web |
| `spring.application.name` | `eureka-server` | Nombre con el que aparece en el panel |
| `eureka.client.register-with-eureka` | `false` | El servidor **no** se registra a sí mismo |
| `eureka.client.fetch-registry` | `false` | El servidor **no** consume el registro |
| `eureka.server.enable-self-preservation` | `false` | Solo dev: no se "protege" a las instancias caída, se expulsan de inmediato |
| `eureka.server.eviction-interval-timer-in-ms` | `5000` | Cadencia de expulsión de instancias inválidas (5 s) |

> La autoprotección desactivada es una decisión **de desarrollo**: en un entorno real (AWS) debe
> estar en `true` para evitar expulsar instancias sanas por una micro-interrupción de red.

## 5. Ciclo de vida de los clientes

| Paso | Quién lo ejecuta | Valor por defecto |
|---|---|---|
| Registro | cada cliente al arrancar (`POST /eureka/apps/{app}`) | — |
| Latido (renew) | cada cliente (`PUT /eureka/apps/{app}/{id}`) | 30 s |
| Consulta del registro | cada cliente (`eureka.client.registry-fetch-interval-seconds`) | 30 s |
| Expulsión | el servidor, si no llegan latidos | según `eviction-interval-timer-in-ms` |

Clientes registrados actualmente:

| Servicio | Nombre en Eureka | Puerto |
|---|---|---|
| API Gateway | `api-gateway` | 8080 |
| Usuarios | `ticketflow-user-service` | 8081 |
| Inventario | `ticketflow-inventory-service` | 8082 |

Los clientes se configuran con `eureka.client.enabled=${EUREKA_ENABLED:true}` y
`eureka.client.service-url.defaultZone=http://localhost:8761/eureka` (en el gateway, la variable es
`EUREKA_URI`).

## 6. Endpoints

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

El detalle de códigos de respuesta y el contraste con la especificación están en
`docs_ia/contexto/contrato_api.md`.

## 7. Observabilidad y operación

- El panel en `http://localhost:8761` muestra instancias, estado y latidos; lo sirve el propio jar
  (no hay HTML en el repo).
- El registro **no es persistente**: un reinicio lo vacía. Los clientes se re-registran solos, pero
  hay una ventana sin resolución de nombres.
- En el despliegue en AWS el registro puede sustituirse por AWS Cloud Map sin tocar el código de
  negocio de los servicios (decisión consciente del curso).

## 8. Desviaciones respecto de la especificación

| Tema | Especificación | Implementado |
|---|---|---|
| Seguridad | «Protegido con Basic Auth» | Sin autenticación (aceptable solo en local) |
| Autoprotección | Activada en producción | Desactivada globalmente (debería ir por perfil de entorno) |
| Persistencia | No especificado | Solo en memoria |
| **Java** | **25 LTS** (estándar del proyecto) | 21 ⚠️ |
| Spring Boot | 4.1.1 (estándar del proyecto) | 4.1.1 ✔ |
| Prefijo `/api/` | Obligatorio para endpoints de negocio | Exento: solo expone API de framework |

Detalle en `docs_ia/contexto/contrato_api.md` (sección 7).
