# Contrato de API — ticketflow-eureka-server

## 1. Propósito y estado

Define el contrato del **registro de servicios**: endpoints de administración, política de autoprotección y ciclo de vida de las instancias registradas.

- **Fuentes de verdad del contrato:**
  - Especificación de plataforma: `docs/Microservicios.docx` (sección 3 «eureka-server») y `docs/Arquitectura.docx` (secciones 2 y 6), copiadas en el workspace raíz.
  - Implementación real: `src/main/resources/application.yml` y el starter `spring-cloud-starter-netflix-eureka-server`.
- **Alcance:** endpoints que expone el propio Eureka. No expone endpoints de negocio.
- **Estado:** vigente. Las desviaciones respecto de la especificación están listadas en la sección 7.

## 2. Identidad del servicio

| Campo | Implementado | Contrato (spec) | Nota |
|---|---|---|---|
| Nombre en Eureka | `eureka-server` | `eureka-server` | Coincide. |
| Puerto | **8761** | 8761 | Coincide. |
| Prefijo de rutas | `/` y `/eureka/**` — **exentos** (ver §5) | `/api/` obligatorio para endpoints de negocio | Exentos: los endpoints los provee el framework. |
| Base de datos | ninguna (registro en memoria) | ninguna | El registro no sobrevive a un reinicio. |
| Stack | Java 21, Spring Boot 4.1.1, Spring Cloud 2025.1.3, Spring Cloud Netflix Eureka Server 5.0.x | Java 25 LTS, Spring Boot 4.1.1, Spring Cloud 2025.1.3 (Netflix 5.0.x) | Coincide salvo la versión de Java. |

## 3. Estándar de versiones

Versiones **obligatorias y uniformes para todo el proyecto**. Solo se listan aquí las tecnologías
que **este servicio usa**; las de los demás repos están en sus respectivos `contrato_api.md`.

| Tecnología | Versión estándar | Versión actual en el repo | Estado |
|---|---|---|---|
| Java | **25 (LTS)** | 21 | ⚠️ Pendiente de migración |
| Spring Boot | **4.1.1** | 4.1.1 | ✔ Al día |
| Spring Cloud | **2025.1.3** | 2025.1.3 | ✔ Al día |
| Spring Cloud Netflix Eureka Server | **5.0.x** | 5.0.x (vía `spring-cloud-starter-netflix-eureka-server`) | ✔ Al día |

> Este servicio **no** usa PostgreSQL, Redis, Kafka, jjwt ni Flyway: no aparecen en su stack.
> PostgreSQL lo usan `ticketflow-user-service` y `ticketflow-inventory-service`; Redis, el
> gateway y el inventory; Kafka, solo el inventory; jjwt, el gateway y el user-service.

## 4. Configuración actual (claves que importan)

| Clave | Valor | Significado |
|---|---|---|
| `server.port` | `8761` | Puerto del registro y del panel. |
| `spring.application.name` | `eureka-server` | Nombre con el que aparece en el panel. |
| `eureka.client.register-with-eureka` | `false` | El servidor no se registra a sí mismo. |
| `eureka.client.fetch-registry` | `false` | El servidor no consume el registro. |
| `eureka.server.enable-self-preservation` | `false` | **Solo en dev**: evita que un servicio caído quede «protegido» y no se expulse. En producción debería ir en `true`. |
| `eureka.server.eviction-interval-timer-in-ms` | `5000` | Cadencia de expulsión de instancias inválidas (5 s). |

Clientes (user-service, inventory-service, api-gateway): `eureka.client.enabled=${EUREKA_ENABLED:true}` y `eureka.client.service-url.defaultZone=${EUREKA_DEFAULT_ZONE:http://localhost:8761/eureka}` (en el gateway, `${EUREKA_URI:...}`).

## 5. Prefijo de rutas `/api/` — exención

> **Norma del proyecto:** todo endpoint de **negocio** de la plataforma debe vivir bajo el prefijo
> **`/api/`**.

Este servicio es una **excepción justificada**: no expone endpoints de negocio. Todos los que
publica son provistos por el framework `spring-cloud-starter-netflix-eureka-server` y por el
panel web de Eureka:

| Endpoint | Proprietario | ¿Se puede prefijar con `/api/`? |
|---|---|---|
| `/` (panel web) | Eureka Server (framework) | No: es el dashboard del framework. |
| `/eureka/apps/**` | Eureka Server (framework) | No: es el contrato de registro que consumen los clientes Eureka (`spring-cloud-starter-netflix-eureka-client`) y el gateway. Prefijarlo rompería el descubrimiento de servicios. |
| `/actuator/**` | Spring Boot (framework) | No: exento por definición (infraestructura). |

**Consecuencia:** el gateway y los demás clientes siguen hablando con Eureka en `/eureka/apps`;
la norma `/api/` aplica a los **endpoints de negocio**, no a las APIs de framework. Cuando se
añada a este servicio algún endpoint propio (por ejemplo, de administración o de métricas de
registro), ese endpoint **sí** debe ir bajo `/api/`.

## 6. Endpoints

### 6.1 Propios del servicio

| Método | Ruta | Rol | Descripción | Respuestas |
|---|---|---|---|---|
| GET | `/` | Público | Panel web de Eureka (instancias, estado, latidos). | 200 (HTML) |
| GET | `/eureka/apps` | Público | Instancias registradas. Acepta `Accept: application/json` o `application/xml`. | 200 |
| GET | `/eureka/apps/{app}` | Público | Instancias de una aplicación. | 200 / 404 |
| POST | `/eureka/apps/{app}` | Cliente Eureka | Registro de una instancia (lo usa el cliente al arrancar). | 204 / 400 |
| PUT | `/eureka/apps/{app}/{id}` | Cliente Eureka | Latido (renovación de lease). | 200 / 404 |
| GET | `/eureka/apps/{app}/{id}` | Cliente Eureka | Latido de una instancia concreta. | 200 / 404 |
| DELETE | `/eureka/apps/{app}/{id}` | Cliente Eureka | Baja de la instancia. | 200 / 404 |
| POST | `/eureka/apps/{app}/{id}/status?value=OUT_OF_SERVICE` | Cliente Eureka | Marca la instancia como no disponible sin darla de baja. | 200 |

### 6.2 Contrato de la especificación

La spec (sección 3) describe el mismo conjunto de endpoints: `GET /`, `GET /eureka/apps`, `POST /eureka/apps/{app}`, `PUT /eureka/apps/{app}/{id}` y `DELETE /eureka/apps/{app}/{id}`. **Coincide** con lo implementado.

## 7. Desviaciones respecto de la especificación

| # | Tema | Especificación | Implementado | Impacto |
|---|---|---|---|---|
| 1 | Seguridad | «Protegido con Basic Auth» | Sin autenticación | El panel y el registro son accesibles sin credenciales. Solo aceptable en local; en AWS debe habilitarse. |
| 2 | Autoprotección | Activada (comportamiento por defecto en AWS) | Desactivada en dev | Riesgo de expulsar instancias sanas por una micro-interrupción de red. Debe activarse por perfil de entorno. |
| 3 | Persistencia del registro | No especificado | Solo en memoria | Un reinicio de Eureka deja el registro vacío; los servicios se re-registran solos, pero hay una ventana sin resolución de nombres. |
| 4 | Sustitución en AWS | Puede reemplazarse por AWS Cloud Map sin tocar código de negocio | Se mantiene Eureka | Es la decisión consciente del curso. |
| 5 | **Java** | **25 LTS** | **21** | Sin hilos virtuales. |
| 6 | Spring Boot | 4.1.1 | 4.1.1 | ✔ Al día |
| 7 | Spring Cloud | 2025.1.3 | 2025.1.3 | ✔ Al día |
| 8 | Prefijo `/api/` | Obligatorio para endpoints de negocio | Exento (solo expone API de framework) | Sin impacto: no hay endpoints de negocio en este servicio. Ver sección 5. |

## 8. Cómo mantener este documento

1. Cambiar la configuración del registro (autoprotección, eviction, puertos) implica actualizar la sección 4.
2. Añadir un cliente de Eureka implica registrar su nombre y puerto en el panel y en el `contrato_api.md` del gateway.
3. Añadir un endpoint propio a este servicio obliga a publicarlo bajo `/api/` (sección 5).
4. La especificación de la plataforma vive fuera de este repositorio (`docs/Microservicios.docx` en la raíz del workspace); aquí solo se refleja su efecto en este servicio.
