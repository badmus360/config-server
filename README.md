POSTMAN COLLECTION - https://drive.google.com/file/d/1ErHQaNppJrYdJ9pn0Y0-9gfAK9lPuO21/view?usp=sharing


MICROSERVICES INFRASTRUCTURE SUITE

OVERVIEW
This system includes three foundational components that support microservices architecture:

1. Config Server - Provides centralized configuration using Spring Cloud Config
2. Discovery Service - Provides service registration and lookup via Eureka
3. API Gateway - Routes and aggregates requests via Spring Cloud Gateway

-------------------------------
1. CONFIG SERVER (PORT: 8882)
-------------------------------

PROFILE: native (file system based)
SECURITY: HTTP Basic Auth (username: configUser, password: configPass)

FEATURES:
- Centralized configuration for all services
- Native file profile (reads from /files directory)
- Spring Boot Actuator endpoints for health/info/refresh
- Works with Spring Cloud Config Client

ENDPOINTS:
- /{application}/{profile}             (AUTH: YES)   -> Fetch configuration
- /actuator/health                     (AUTH: NO)    -> Health check
- /actuator/info                       (AUTH: NO)    -> Info
- /actuator/refresh                    (AUTH: YES)   -> Refresh configs

EXAMPLE REQUEST:
GET /discovery/default HTTP/1.1
Authorization: Basic Y29uZmlnVXNlcjpjb25maWdQYXNz

EXAMPLE RESPONSE:
{
  "name": "discovery",
  "profiles": ["default"],
  "version": "1.0",
  "propertySources": [
    {
      "name": "classpath:/files/discovery.yml",
      "source": {
        "server.port": 8761,
        "spring.application.name": "discovery"
      }
    }
  ]
}

--------------------------------
2. DISCOVERY SERVICE (PORT: 8761)
--------------------------------

SECURITY: HTTP Basic Auth (username: switchUser, password: password123)

FEATURES:
- Eureka-based registry and dashboard
- Services register themselves dynamically
- Exposes Spring Boot Actuator for health/metrics
- Self-preservation mode disabled for rapid failover

ENDPOINTS:
- /eureka/**                          (AUTH: YES)    -> Dashboard & registry APIs
- /actuator/health                    (AUTH: NO)     -> Health check
- /actuator/info                      (AUTH: NO)     -> Application info
- /actuator/metrics                   (AUTH: NO)     -> Performance metrics

EXAMPLE REQUEST:
GET /eureka/apps HTTP/1.1
Authorization: Basic c3dpdGNoVXNlcjpwYXNzd29yZDEyMw==

EXAMPLE RESPONSE:
{
  "applications": {
    "versions__delta": "1",
    "apps__hashcode": "UP_1_",
    "application": [
      {
        "name": "GATEWAY",
        "instance": [
          {
            "hostName": "localhost",
            "app": "GATEWAY",
            "ipAddr": "127.0.0.1",
            "status": "UP",
            "port": {"$": 7899, "@enabled": "true"}
          }
        ]
      }
    ]
  }
}

-----------------------------
3. API GATEWAY (PORT: 7899)
-----------------------------

SECURITY: API Key Authentication
HEADER: X-API-Key: secure-key-12345

FEATURES:
- Spring Cloud Gateway
- Integrated with Eureka for dynamic service resolution
- Defined routing paths for each microservice
- Actuator endpoints for health/metrics
- Virtual thread security context enabled

ROUTING TABLE:
- /transfer/**   => http://localhost:8099
- /user/**       => http://localhost:8096
- /identity/**   => http://localhost:8091
- /bills/**      => http://localhost:8090

EXAMPLE REQUEST:
GET /user/profile/123 HTTP/1.1
X-API-Key: secure-key-12345

EXAMPLE RESPONSE:
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}

--------------------------
CONFIGURATION MANAGEMENT
--------------------------

CONFIG FILE DIRECTORY:
src/main/resources/files/

REQUIRED FILES:
- discovery.yml  -> Discovery service settings
- gateway.yml    -> Gateway route configs
- application.yml -> Common settings for all services

------------------
RUNNING THE SYSTEM
------------------

RUN SERVICES IN THIS ORDER:

1. CONFIG SERVER
java -jar configserver.jar

2. DISCOVERY SERVICE
java -jar discovery.jar

3. API GATEWAY
java -jar gateway.jar

VERIFY SYSTEM:
- Config Server:    http://localhost:8882/actuator/health
- Discovery Server: http://localhost:8761/eureka/
- API Gateway:      http://localhost:7899/actuator/health

--------------------------
ACTUATOR MONITORING ROUTES
--------------------------

- /actuator/health   -> Health status
- /actuator/info     -> Basic info
- /actuator/metrics  -> Gateway runtime metrics

-----------------------
SECURITY CONSIDERATIONS
-----------------------

- Use strong credentials and rotate API keys
- Replace default users/passwords in production
- Add HTTPS (TLS/SSL) to all public endpoints
- CSRF disabled for Eureka endpoints
- Expose only health/info metrics publicly
- Encrypt passwords using BCrypt in config files

------------------
TECH STACK / DEPENDENCIES
------------------

- Java 17+
- Spring Boot 3.x
- Spring Cloud Config Server
- Spring Cloud Netflix Eureka
- Spring Cloud Gateway
- Spring Security
- Reactor Netty
