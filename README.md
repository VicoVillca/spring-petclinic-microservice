
## 📋 Descripción del Proyecto

Spring PetClinic REST es una API RESTful desarrollada con Spring Boot para la gestión administrativa de una clínica veterinaria. El proyecto implementa un pipeline completo de **Integración Continua y Entrega Continua (CI/CD)** utilizando:

- **Jenkins**: Automatización del pipeline
- **SonarQube**: Análisis de calidad de código
- **Docker**: Contenedorización de la aplicación
- **DockerHub**: Repositorio de imágenes Docker

## 🎯 Funcionalidades

### API REST
- **Veterinarios**: CRUD completo, búsqueda por especialidad
- **Dueños**: Registro, actualización, historial
- **Mascotas**: Registro, asignación a dueños, tipos
- **Visitas**: Registro de atención, historial por mascota

### Pipeline CI/CD
- ✅ Compilación automática
- ✅ Ejecución de pruebas unitarias
- ✅ Reporte de cobertura (JaCoCo)
- ✅ Análisis de calidad (SonarQube)
- ✅ Generación de FAT JAR
- ✅ Construcción de imagen Docker
- ✅ Publicación en DockerHub


## 🚀 Guía de Configuración CI/CD

### Prerrequisitos

1. **Instancias AWS** (o servidores locales):

   - Jenkins: http://3.91.34.10:8080
   - SonarQube: http://3.91.34.10:9000

2. **Cuentas requeridas**:

   - DockerHub - Cuenta gratuita
   - GitHub - Repositorio público

3. **Repositorio GitHub**:

   - URL: https://github.com/VicoVillca/spring-petclinic-microservice.git
   - Rama: main

---

### PASO 1: Configurar Jenkins

#### 1.1 Acceder a Jenkins

URL: http://3.91.34.10:8080

#### 1.2 Instalar Plugins Necesarios

Ir a: Manage Jenkins → Plugins → Available plugins

Instalar:
- ✅ Docker Pipeline
- ✅ Docker Commons
- ✅ SonarQube Scanner
- ✅ Git
- ✅ Pipeline: Stage View

#### 1.3 Configurar Docker en Jenkins

Verificar que Docker está instalado en el servidor Jenkins:

docker --version
docker ps

Agregar usuario Jenkins al grupo docker:

sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

#### 1.4 Configurar SonarQube en Jenkins

Ir a: Manage Jenkins → Configure System

Buscar SonarQube servers:

Name: sonarqube
Server URL: http://3.91.34.10:9000
Server authentication token: [crear credencial]

Para crear el token en SonarQube:

1. Ve a http://3.91.34.10:9000
2. Login: admin / admin
3. Click en tu avatar → My Account → Security
4. Generar token con nombre jenkins-token
5. Copiar el token

En Jenkins - Credencial del token:

Kind: Secret text
Scope: Global
Secret: [pegar el token de SonarQube]
ID: sonar-token
Description: SonarQube Token

#### 1.5 Configurar Credenciales de DockerHub

Ir a: Manage Jenkins → Credentials → Global

Agregar credenciales de DockerHub:

Kind: Username with password
Scope: Global
Username: sinvidasocial
Password: [tu token de acceso de DockerHub]
ID: dockerhub-credentials
Description: DockerHub Credentials

---

### PASO 2: Configurar SonarQube

#### 2.1 Acceder a SonarQube

URL: http://3.91.34.10:9000

#### 2.2 Crear Proyecto

1. Click en Projects → Create project
2. Seleccionar Manually
3. Project key: spring-petclinic-rest
4. Display name: Spring PetClinic REST
5. Click en Create

#### 2.3 Configurar Quality Gate

1. Ir a Quality Gates
2. Seleccionar Sonar way (por defecto)
3. Verificar condiciones:
   - Coverage < 80% → Error
   - Duplications > 3% → Error

---

### PASO 3: Configurar Docker

#### 3.1 Crear Dockerfile

El Dockerfile ya está incluido en el repositorio:

FROM eclipse-temurin:21-alpine
WORKDIR /workspace
COPY target/spring-petclinic-rest-*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

#### 3.2 Verificar Localmente

# Construir imagen
docker build -t sinvidasocial/spring-petclinic-rest:latest .

# Ejecutar contenedor
docker run -p 8080:8080 sinvidasocial/spring-petclinic-rest:latest

# Verificar
curl http://localhost:8080/actuator/health

---

### PASO 4: Crear Pipeline en Jenkins

#### 4.1 Nuevo Pipeline

1. En Jenkins: New Item
2. Nombre: spring-petclinic-microservice
3. Tipo: Pipeline
4. Click en OK

#### 4.2 Configurar Pipeline

General:
- ✅ GitHub project
- Project URL: https://github.com/VicoVillca/spring-petclinic-microservice

Build Triggers:
- ✅ GitHub hook trigger for GITScm polling

Pipeline:

Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://github.com/VicoVillca/spring-petclinic-microservice.git
Credentials: github-credentials
Branches to build: */main
Script Path: Jenkinsfile

#### 4.3 Guardar y Ejecutar

1. Click en Save
2. Click en Build Now
3. Ver el progreso en Console Output

---

### PASO 5: Verificar Resultados

#### 5.1 En Jenkins

- Ver logs del build: Console Output
- Ver estado de cada stage en la vista de pipeline

#### 5.2 En SonarQube

URL: http://3.91.34.10:9000/dashboard?id=spring-petclinic-rest


#### 5.3 En DockerHub

URL: https://hub.docker.com/r/sinvidasocial/spring-petclinic-rest

Verificar:
- ✅ Imagen sinvidasocial/spring-petclinic-rest:4.0.8
- ✅ Imagen sinvidasocial/spring-petclinic-rest:latest

#### 5.4 Probar la Imagen

# Descargar la imagen
docker pull sinvidasocial/spring-petclinic-rest:latest

# Ejecutar el contenedor
docker run -p 8080:8080 sinvidasocial/spring-petclinic-rest:latest

# Verificar
curl http://localhost:8080/actuator/health

---

## 📂 Estructura del Proyecto

```
spring-petclinic-rest/
├── src/
│   ├── main/
│   │   ├── java/          # Código fuente
│   │   └── resources/     # Configuraciones
│   └── test/
│       └── java/          # Pruebas unitarias
├── target/                # Artefactos generados
├── Jenkinsfile            # Pipeline CI/CD (¡archivo principal!)
├── pom.xml                # Dependencias Maven
└── README.md              # Documentación
```


---

## 🔧 Pipeline CI/CD (Jenkinsfile)

El pipeline incluye las siguientes etapas:

```
stages {
    stage(Compile)          // Compilación del código
    stage(Test)             // Ejecutamos Test
    stage(Coverage)         // Reporte de covertura
    stage(SonarQube)        // Analizamos codigo
    stage(Package)          // Generación de FAT JAR
    stage(DockerHub)        // Build y push a DockerHub
}
```

---

## 🐳 Imagen Docker

### Información de la Imagen

- Usuario: sinvidasocial
- Repositorio: spring-petclinic-rest
- Tags: 4.0.8, latest
- URL: https://hub.docker.com/r/sinvidasocial/spring-petclinic-rest

### Comandos Útiles

# Descargar la imagen
docker pull sinvidasocial/spring-petclinic-rest:latest

# Ejecutar el contenedor
docker run -d -p 8080:8080 --name petclinic sinvidasocial/spring-petclinic-rest:latest

# Ver logs
docker logs -f petclinic

# Detener el contenedor
docker stop petclinic

# Eliminar el contenedor
docker rm petclinic

# Verificar estado
curl http://localhost:8080
---


## 👤 Autor

VicoVillca

- GitHub: https://github.com/VicoVillca
- DockerHub: https://hub.docker.com/r/sinvidasocial

---