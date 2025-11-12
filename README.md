# Payment Chain - Microservices Architecture

Sistema de microservicios para la gestión de transacciones, clientes y productos bancarios basado en Spring Boot y Spring Cloud.

## Arquitectura del Sistema

Este proyecto implementa una arquitectura de microservicios completa con los siguientes componentes:

### Microservicios de Negocio (businessdomain)

- **Customer Service** (Puerto 8081)
  - Gestión de clientes y sus productos asociados
  - Base de datos: H2 (desarrollo)
  - Entidades: Customer, CustomerProduct

- **Product Service** (Puerto 8082)
  - Catálogo de productos financieros
  - Base de datos: H2 (desarrollo)
  - Entidades: Product

- **Transaction Service** (Puerto 8083)
  - Gestión de transacciones bancarias
  - Base de datos: H2 (desarrollo)
  - Entidades: Transaction, Channel, Status
  - Incluye: Spring Security, MapStruct para DTOs

### Servicios de Infraestructura (infraestructuradomain)

- **Config Server** (Puerto 8888)
  - Servidor de configuración centralizada
  - Backend: Repositorio Git local (config-server-repo)
  - Perfiles: dev, qa, prod

- **Eureka Server** (Puerto 8761)
  - Service Discovery y registro de servicios
  - Dashboard disponible en http://localhost:8761

- **API Gateway** (Puerto 8080)
  - Puerta de entrada única al sistema
  - Spring Cloud Gateway con Circuit Breaker
  - Rate Limiting configurado
  - Punto de acceso principal: http://localhost:8080

- **Spring Boot Admin** (Puerto 8762)
  - Monitoreo y administración de microservicios
  - Dashboard disponible en http://localhost:8762

## Stack Tecnológico

- **Java**: 17
- **Spring Boot**: 3.2.2
- **Spring Cloud**: 2023.0.0
- **Build Tool**: Maven 3.9.9
- **Service Discovery**: Netflix Eureka
- **API Gateway**: Spring Cloud Gateway
- **Configuration**: Spring Cloud Config
- **Monitoring**: Spring Boot Admin 3.4.5
- **Database**: H2 (desarrollo), PostgreSQL (producción)
- **Containerization**: Docker & Docker Compose
- **Libraries**: Lombok, MapStruct, WebFlux

## Requisitos Previos

- **Java JDK 17** o superior
- **Maven 3.6+**
- **Docker** y **Docker Compose** (opcional, para ejecución en contenedores)
- **Git** (para clonar el repositorio)

## Instalación y Configuración

### 1. Clonar el Repositorio

```bash
git clone <repository-url>
cd paymentchainparent
```

### 2. Compilar el Proyecto

Compilar todos los módulos:

```bash
mvn clean install
```

Compilar sin tests:

```bash
mvn clean install -DskipTests
```

### 3. Orden de Ejecución de Servicios

Es importante iniciar los servicios en el siguiente orden:

#### Paso 1: Config Server
```bash
cd infraestructuradomain/configserver
mvn spring-boot:run
```
Esperar a que inicie completamente (puerto 8888)

#### Paso 2: Eureka Server
```bash
cd infraestructuradomain/eurekaServer
mvn spring-boot:run
```
Esperar a que inicie y se registre (puerto 8761)

#### Paso 3: Spring Boot Admin (opcional)
```bash
cd infraestructuradomain/springBootAdmin
mvn spring-boot:run
```

#### Paso 4: Microservicios de Negocio

En terminales separadas:

```bash
# Customer Service
cd businessdomain/customer
mvn spring-boot:run

# Product Service
cd businessdomain/product
mvn spring-boot:run

# Transaction Service
cd businessdomain/transaction
mvn spring-boot:run
```

#### Paso 5: API Gateway
```bash
cd infraestructuradomain/apigetway
mvn spring-boot:run
```

## Ejecución con Docker

### Construir Imágenes

```bash
# Compilar JARs primero
mvn clean package -DskipTests

# Construir todas las imágenes
docker-compose build
```

### Iniciar Todos los Servicios

```bash
docker-compose up -d
```

### Ver Logs

```bash
# Todos los servicios
docker-compose logs -f

# Servicio específico
docker-compose logs -f customer
```

### Detener Servicios

```bash
docker-compose down
```

## Endpoints Principales

### Acceso a través del API Gateway (Puerto 8080)

- **Customer Service**: http://localhost:8080/customer/
- **Product Service**: http://localhost:8080/product/
- **Transaction Service**: http://localhost:8083/ (directo, configurar ruta en gateway)

### Acceso Directo (Desarrollo)

- **Customer Service**: http://localhost:8081
  - Swagger UI: http://localhost:8081/swagger-ui.html
  - API Docs: http://localhost:8081/v3/api-docs

- **Product Service**: http://localhost:8082
  - Swagger UI: http://localhost:8082/swagger-ui.html

- **Transaction Service**: http://localhost:8083
  - Swagger UI: http://localhost:8083/swagger-ui.html

### Servicios de Infraestructura

- **Eureka Dashboard**: http://localhost:8761
- **Config Server**: http://localhost:8888
- **Spring Boot Admin**: http://localhost:8762

## Configuración por Ambientes

Las configuraciones están centralizadas en `config-server-repo/`:

```
config-server-repo/
├── application.yml              # Configuración global
├── config-client.yml            # Configuración de clientes
└── environments/
    ├── dev/                     # Desarrollo
    │   ├── businessdomain-customer.yml
    │   ├── businessdomain-product.yml
    │   └── businessdomain-transaction.yml
    ├── qa/                      # Quality Assurance
    └── prod/                    # Producción
```

### Activar un Perfil

```bash
# Mediante variable de entorno
export SPRING_PROFILES_ACTIVE=dev
mvn spring-boot:run

# O mediante argumento
mvn spring-boot:run -Dspring-boot.run.arguments=--spring.profiles.active=dev
```

## Ejemplos de Uso

### Crear un Cliente

```bash
curl -X POST http://localhost:8080/customer/ \
  -H "Content-Type: application/json" \
  -d '{
    "name": "John Doe",
    "phone": "555-1234",
    "address": "123 Main St"
  }'
```

### Crear un Producto

```bash
curl -X POST http://localhost:8080/product/ \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Savings Account",
    "code": "SAV001"
  }'
```

### Consultar Clientes

```bash
curl http://localhost:8080/customer/
```

Consulta el archivo [info.txt](info.txt) para más ejemplos de datos de prueba.

## Estructura del Proyecto

```
paymentchainparent/
├── businessdomain/                 # Microservicios de dominio
│   ├── customer/                  # Servicio de clientes
│   ├── product/                   # Servicio de productos
│   └── transaction/               # Servicio de transacciones
├── infraestructuradomain/         # Servicios de infraestructura
│   ├── configserver/             # Servidor de configuración
│   ├── eurekaServer/             # Service discovery
│   ├── apigetway/                # API Gateway
│   └── springBootAdmin/          # Admin dashboard
├── config-server-repo/           # Configuraciones centralizadas
├── docker-compose.yml            # Orquestación de contenedores
├── pom.xml                       # POM padre
└── README.md                     # Este archivo
```

## Desarrollo

### Agregar un Nuevo Microservicio

1. Crear módulo en `businessdomain/` o `infraestructuradomain/`
2. Agregar dependencia en el POM padre
3. Configurar puerto único
4. Crear configuración en `config-server-repo/`
5. Registrar en Eureka
6. Agregar rutas en API Gateway (si es necesario)
7. Crear Dockerfile

### Tests

Ejecutar todos los tests:

```bash
mvn test
```

Ejecutar tests de un módulo específico:

```bash
cd businessdomain/customer
mvn test
```

### Generar Documentación

```bash
mvn site
mvn javadoc:aggregate
```

La documentación se generará en `target/site/`

## Monitoreo y Administración

### Spring Boot Admin

Accede a http://localhost:8762 para:
- Ver estado de todos los microservicios
- Métricas en tiempo real
- Información de salud (health checks)
- Variables de entorno
- Logs en tiempo real

### Eureka Dashboard

Accede a http://localhost:8761 para:
- Ver servicios registrados
- Estado de las instancias
- Información de registro

## Troubleshooting

### El servicio no se registra en Eureka

- Verificar que Eureka Server esté iniciado
- Revisar la configuración en `bootstrap.yml` o `application.yml`
- Verificar conectividad de red
- Revisar logs: `mvn spring-boot:run` o `docker-compose logs`

### Config Server no encuentra configuraciones

- Verificar que el repositorio Git local esté inicializado
- Verificar la ruta del repositorio en `application.yml`
- Asegurar que los archivos YAML estén nombrados correctamente

### Problemas de memoria en Docker

Ajustar la memoria JVM en `docker-compose.yml`:

```yaml
environment:
  JAVA_OPTS: "-Xms256M -Xmx512M"
```

### Puerto ya en uso

```bash
# Windows
netstat -ano | findstr :8080
taskkill /PID <PID> /F

# Linux/Mac
lsof -i :8080
kill -9 <PID>
```

## Mejores Prácticas

1. **Orden de inicio**: Siempre iniciar Config Server y Eureka primero
2. **Configuración sensible**: Usar variables de entorno para credenciales
3. **Logging**: Configurar niveles apropiados por ambiente
4. **Health checks**: Implementar en todos los microservicios
5. **Circuit breakers**: Usar Resilience4j para tolerancia a fallos
6. **Versionado**: Versionar APIs desde el inicio

## Contribuir

1. Fork el proyecto
2. Crear una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Commit tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abrir un Pull Request

## Licencia

Este proyecto es de uso educativo y de desarrollo.

## Contacto

Para preguntas o soporte, consulta la documentación de cada módulo o los archivos HELP.md individuales.

## Referencias

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
- [Spring Cloud Gateway](https://spring.io/projects/spring-cloud-gateway)
- [Netflix Eureka](https://github.com/Netflix/eureka)
- [Spring Boot Admin](https://github.com/codecentric/spring-boot-admin)
