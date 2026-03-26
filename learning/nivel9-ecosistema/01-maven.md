# 9.1 Maven

Maven es la herramienta de construcción y gestión de dependencias más usada
en proyectos Java. Automatiza la compilación, testing, empaquetado y despliegue.

## Estructura de un proyecto Maven

```
mi-proyecto/
├── pom.xml                          ← Configuración del proyecto
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/escuela/
│   │   │         └── Main.java
│   │   └── resources/
│   │         └── application.properties
│   └── test/
│       ├── java/
│       │   └── com/escuela/
│       │         └── MainTest.java
│       └── resources/
└── target/                          ← Archivos compilados (generado por Maven)
    └── mi-proyecto-1.0.jar
```

---

## El POM (pom.xml)

El POM (Project Object Model) es el archivo de configuración central de Maven.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!-- Coordenadas del proyecto (GAV) -->
    <groupId>com.escuela</groupId>        <!-- Organización / paquete base -->
    <artifactId>sistema-escolar</artifactId> <!-- Nombre del proyecto -->
    <version>1.0.0-SNAPSHOT</version>    <!-- Versión (SNAPSHOT = en desarrollo) -->
    <packaging>jar</packaging>           <!-- jar, war, pom -->

    <!-- Metadata opcional -->
    <name>Sistema de Gestión Escolar</name>
    <description>Aplicación para gestionar estudiantes, cursos y calificaciones</description>

    <!-- Propiedades configurables -->
    <properties>
        <java.version>17</java.version>
        <maven.compiler.source>${java.version}</maven.compiler.source>
        <maven.compiler.target>${java.version}</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>

        <!-- Versiones de dependencias centralizadas -->
        <junit.version>5.10.0</junit.version>
        <mockito.version>5.5.0</mockito.version>
    </properties>

    <!-- Dependencias -->
    <dependencies>
        <!-- JUnit 5 para testing -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>    <!-- solo para compilar/ejecutar tests -->
        </dependency>

        <!-- Mockito -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>${mockito.version}</version>
            <scope>test</scope>
        </dependency>

        <!-- Base de datos en memoria para tests -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.2.224</version>
            <scope>test</scope>
        </dependency>

        <!-- Logging -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.9</version>
        </dependency>
    </dependencies>

    <!-- Configuración de construcción -->
    <build>
        <plugins>
            <!-- Plugin para compilar con Java 17 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                </configuration>
            </plugin>

            <!-- Plugin para ejecutar tests de JUnit 5 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.1.2</version>
            </plugin>

            <!-- Crear un JAR ejecutable con todas las dependencias -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>com.escuela.Main</mainClass>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals><goal>single</goal></goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

---

## Ciclo de Vida Maven

```bash
# Compilar código fuente
mvn compile

# Ejecutar tests
mvn test

# Compilar + test + empaquetar en JAR
mvn package

# Instalar en repositorio local (~/.m2)
mvn install

# Limpiar archivos generados (carpeta target/)
mvn clean

# Limpiar + compilar + test + empaquetar
mvn clean package

# Saltar tests (no recomendado, pero a veces necesario)
mvn package -DskipTests

# Ver árbol de dependencias
mvn dependency:tree

# Buscar dependencias desactualizadas
mvn versions:display-dependency-updates
```

### Las fases del ciclo

```
validate → compile → test → package → verify → install → deploy
```

Cada fase incluye automáticamente todas las anteriores.
`mvn package` ejecuta: validate, compile, test, package.

---

## Scopes de dependencias

| Scope | Disponible en | Incluido en JAR | Uso típico |
|-------|--------------|----------------|-----------|
| `compile` (default) | Todo | Sí | Librerías de la app |
| `test` | Solo tests | No | JUnit, Mockito |
| `provided` | Compilación y test | No | Servlet API (provista por servidor) |
| `runtime` | Ejecución y test | Sí | Driver JDBC |
| `system` | Compilación | No | JARs locales |

---

## Herencia con POM padre (Parent POM)

```xml
<!-- pom.xml de módulo hijo -->
<parent>
    <groupId>com.escuela</groupId>
    <artifactId>escuela-parent</artifactId>
    <version>1.0.0</version>
</parent>

<!-- Hereda propiedades, dependencias y plugins del padre -->
```

---

## Resumen Maven

| Concepto | Descripción |
|---------|-------------|
| `pom.xml` | Configuración del proyecto |
| GAV | groupId, artifactId, version — identifican un artefacto |
| `mvn clean package` | Limpia y empaqueta |
| `mvn test` | Ejecuta todos los tests |
| `scope` | Controla cuándo está disponible la dependencia |
| Repositorio local | `~/.m2/repository` — caché local de dependencias |

**Siguiente:** [9.2 Spring Boot Introducción](02-spring-boot-intro.md)
