# 🏥 Spring PetClinic REST - CI/CD Pipeline

[![Java](https://img.shields.io/badge/Java-21-red)](https://adoptium.net/)
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.7-green)](https://spring.io/projects/spring-boot)
[![Maven](https://img.shields.io/badge/Maven-3.8.8-yellow)](https://maven.apache.org/)
[![Jenkins](https://img.shields.io/badge/Jenkins-CI%2FCD-blue)](https://www.jenkins.io/)
[![SonarQube](https://img.shields.io/badge/SonarQube-Quality-4E9BCD)](https://www.sonarqube.org/)
[![JFrog](https://img.shields.io/badge/JFrog-Artifactory-6B8E23)](https://jfrog.com/artifactory/)

## 📋 Descripción del Proyecto

Spring PetClinic REST es una API RESTful desarrollada con Spring Boot para la gestión administrativa de una clínica veterinaria. El proyecto implementa un pipeline completo de **Integración Continua y Entrega Continua (CI/CD)** utilizando:

- **Jenkins**: Automatización del pipeline
- **SonarQube**: Análisis de calidad de código
- **JFrog Artifactory**: Repositorio de artefactos

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
- ✅ Generación de artefactos (JAR)
- ✅ Publicación en Artifactory

## 🛠️ Tecnologías Utilizadas

| Tecnología | Versión | Propósito |
|------------|---------|-----------|
| Java | 21 | Lenguaje principal |
| Spring Boot | 2.7 | Framework REST |
| Spring Data JPA | - | Persistencia |
| H2 Database | - | Base de datos en memoria |
| Maven | 3.8.8 | Gestión de dependencias |
| JUnit 5 | - | Pruebas unitarias |
| JaCoCo | - | Cobertura de código |
| **Jenkins** | - | CI/CD Pipeline |
| **SonarQube** | - | Calidad de código |
| **JFrog Artifactory** | - | Repositorio de artefactos |
| **Docker** | - | Contenedorización |

---

## 🚀 Guía de Configuración CI/CD

### Prerrequisitos

1. **Instancias AWS** (o servidores locales):

   - Jenkins: http://3.91.34.10:8080

   - SonarQube: http://3.91.34.10:9000

   - JFrog Artifactory Admin: http://3.91.34.10:8082

   - JFrog Artifactory Para servicios: http://3.91.34.10:8081

2. **Repositorio GitHub**:

   - URL: https://github.com/VicoVillca/spring-petclinic-rest.git

   - Rama: main

---

### PASO 1: Configurar Jenkins

#### 1.1 Acceder a Jenkins

```
URL: http://3.91.34.10:8080
```

#### 1.4 Configurar SonarQube en Jenkins

Ir a: **Manage Jenkins** → **Configure System**

Buscar **SonarQube servers**:

```
Name: sonarqube
Server URL: http://3.91.34.10:9000
Server authentication token: [crear credencial]
```

**Para crear el token en SonarQube**:

1. Ve a http://3.91.34.10:9000

2. Login: admin / admin

3. Click en tu avatar → **My Account** → **Security**

4. Generar token con nombre jenkins-token

5. Copiar el token

**En Jenkins - Credencial del token**:

```
Kind: Secret text
Scope: Global
Secret: [pegar el token de SonarQube]
ID: sonar-token
Description: SonarQube Token
```

#### 1.5 Configurar Artifactory en Jenkins

Ir a: **Manage Jenkins** → **Configure System**

Buscar **Artifactory**:

```
Server ID: artifactory
URL: http://3.91.34.10:8081
Credentials: [usuario/clave de Artifactory]
```

**Credenciales de Artifactory**:

```
Kind: Username with password
Scope: Global
Username: admin
Password: [contraseña de Artifactory]
ID: artifactory-credentials
Description: Artifactory Credentials
```


### PASO 2: Configurar SonarQube

#### 2.1 Acceder a SonarQube

```
URL: http://3.91.34.10:9000
```

#### 2.2 Crear Proyecto

1. Click en **Projects** → **Create project**

2. Seleccionar **Manually**

3. Project key: spring-petclinic-rest

4. Display name: Spring PetClinic REST

5. Click en **Create**

#### 2.3 Configurar Quality Gate

1. Ir a **Quality Gates**

2. Seleccionar **Sonar way** (por defecto)

3. Verificar condiciones:

   - Coverage < 80% → Error

   - Duplications > 3% → Error

---

### PASO 3: Configurar JFrog Artifactory

#### 3.1 Acceder a Artifactory

```
URL: http://3.91.34.10:8082
```

#### 3.2 Crear Repositorios Maven

**Repositorio Release**:

1. Click en **Create Repository**

2. Tipo: **Maven**

3. Repository Key: spring-petclinic-rest-release

4. Click en **Create**

**Repositorio Snapshot**:

1. Click en **Create Repository**

2. Tipo: **Maven**

3. Repository Key: spring-petclinic-rest-snapshot

4. Click en **Create**

#### 3.3 Verificar Permisos

1. Ir a **Administration** → **Permissions**

2. Asegurar que admin tiene permisos de escritura

---

### PASO 4: Crear Pipeline en Jenkins

#### 4.1 Nuevo Pipeline

1. En Jenkins: **New Item**

2. Nombre: spring-petclinic-rest

3. Tipo: **Pipeline**

4. Click en **OK**

#### 4.2 Configurar Pipeline

En la configuración del pipeline:

**General**:

- ✅ GitHub project

- Project URL: https://github.com/VicoVillca/spring-petclinic-rest

**Build Triggers**:

- ✅ GitHub hook trigger for GITScm polling

**Pipeline**:

```
Definition: Pipeline script from SCM
SCM: Git
Repository URL: https://github.com/VicoVillca/spring-petclinic-rest.git
Credentials: github-credentials
Branches to build: */main
Script Path: Jenkinsfile
```

#### 4.3 Guardar y Ejecutar

1. Click en **Save**

2. Click en **Build Now**

3. Ver el progreso en **Console Output**

---

### PASO 5: Verificar Resultados

#### 5.1 En Jenkins

- Ver logs del build: **Console Output**

- Ver artefactos: **Artifacts** del build

#### 5.2 En SonarQube

```
URL: http://3.91.34.10:9000/dashboard?id=spring-petclinic-rest
```

Verificar:

- ✅ Coverage

- ✅ Code Smells

- ✅ Duplications

- ✅ Security Hotspots

#### 5.3 En Artifactory

```
URL: http://3.91.34.10:8082
```

Verificar:

- ✅ spring-petclinic-rest-release/ (versiones estables)

- ✅ spring-petclinic-rest-snapshot/ (versiones en desarrollo)

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
    stage($Compile$)          // Compilación del código
    stage($Test$)             // Ejecución de pruebas
    stage($Coverage$)         // Reporte de cobertura (JaCoCo)
    stage($SonarQube$)        // Análisis de calidad
    stage($Package and Deploy$) // Empaquetado y despliegue a Artifactory
}
```


---

## 👤 Autor

**VicoVillca**

- GitHub: https://github.com/VicoVillca

