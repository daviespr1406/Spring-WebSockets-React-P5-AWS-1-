# 🧩 Proyecto: Interactive Blackboard - Spring Boot, WebSockets, React y P5.js

**Autores:**  
- David Espinosa  

**Curso:** ARSW — Escuela Colombiana de Ingeniería Julio Garavito  
**Fecha:** Octubre 2025  

---

## 📘 Descripción General

Este proyecto implementa una **pizarra interactiva colaborativa** que permite dibujar en tiempo real con múltiples usuarios conectados mediante **WebSockets**.  
Combina **Spring Boot** en el backend con **ReactJS** y **P5.js** en el frontend, aplicando buenas prácticas de diseño para lograr una arquitectura escalable y moderna.  

El video explicativo del proyecto se encuentra en la carpeta:  
📂 `assets/video-demostracion.mp4`

---

## ⚙️ Estructura del Proyecto

```
Spring-WebSockets-React-P5-AWS/
│
├── src/
│   ├── main/
│   │   ├── java/org/example/
│   │   │   ├── BBAppStarter.java
│   │   │   ├── configurator/BBConfigurator.java
│   │   │   └── endpoints/BBEndpoint.java
│   │   └── resources/static/
│   │       └── index.html
│   └── test/
│
├── pom.xml
├── target/
└── assets/
    └── video-demostracion.mp4
```

---

## 🧠 Tecnologías Utilizadas

- **Java 17**
- **Spring Boot 3.1.1**
- **Spring WebSocket**
- **React 18 (UMD)**
- **P5.js 0.7.1**
- **AWS EC2 (para despliegue)**
- **Maven**

---

## 🚀 Pasos para la Construcción y Ejecución

### 1️⃣ Crear el Proyecto Maven
```bash
mvn archetype:generate -DarchetypeGroupId=org.apache.maven.archetypes -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4
```

### 2️⃣ Configurar el `pom.xml`
Incluye las dependencias necesarias:
```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.1.1</version>
  </dependency>

  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
    <version>3.1.1</version>
  </dependency>

  <dependency>
    <groupId>jakarta.websocket</groupId>
    <artifactId>jakarta.websocket-api</artifactId>
    <scope>provided</scope>
  </dependency>
</dependencies>

<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-dependency-plugin</artifactId>
      <version>3.6.0</version>
      <executions>
        <execution>
          <id>copy-dependencies</id>
          <phase>package</phase>
          <goals><goal>copy-dependencies</goal></goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

---

### 3️⃣ Backend – Spring Boot

**Clase principal:**
```java
@SpringBootApplication
public class BBAppStarter {
    public static void main(String[] args){
        SpringApplication app = new SpringApplication(BBAppStarter.class);
        app.setDefaultProperties(Collections.singletonMap("server.port", getPort()));
        app.run(args);
    }

    static int getPort() {
        if (System.getenv("PORT") != null) {
            return Integer.parseInt(System.getenv("PORT"));
        }
        return 8080; // Puerto por defecto local
    }
}
```

**Configurador:**
```java
@Configuration
@EnableScheduling
public class BBConfigurator {
    @Bean
    public ServerEndpointExporter serverEndpointExporter() {
        return new ServerEndpointExporter();
    }
}
```

**Endpoint WebSocket:**
```java
@ServerEndpoint("/bbService")
@Component
public class BBEndpoint {
    private static final Logger logger = Logger.getLogger(BBEndpoint.class.getName());
    static Queue<Session> queue = new ConcurrentLinkedQueue<>();
    Session ownSession = null;

    public void send(String msg) throws IOException {
        for (Session session : queue) {
            if (!session.equals(this.ownSession)) {
                session.getBasicRemote().sendText(msg);
            }
        }
    }

    @OnMessage
    public void processPoint(String message, Session session) {
        this.send(message);
    }

    @OnOpen
    public void openConnection(Session session) throws IOException {
        queue.add(session);
        ownSession = session;
        session.getBasicRemote().sendText("Connection established.");
    }

    @OnClose
    public void closedConnection(Session session) {
        queue.remove(session);
    }
}
```

---

### 4️⃣ Frontend – React + P5.js

**index.html**
```html
<!DOCTYPE html>
<html>
<head>
    <title>Interactive BB</title>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/0.7.1/p5.min.js"></script>
    <script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
    <script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
</head>
<body>
    <div id="root"></div>
</body>
</html>
```

---

### 5️⃣ Compilar y Ejecutar Localmente

```bash
mvn clean package
java -cp target/classes;target/dependency/* org.example.BBAppStarter
```

Abre en tu navegador:
```
http://localhost:8080/
```

---

### 6️⃣ Despliegue en AWS EC2

1. Sube los archivos al servidor usando `scp` o `git clone`.
2. Asegúrate de tener abierto el puerto **8080** en el Security Group.
3. Ejecuta el comando:
   ```bash
   java -cp target/classes:target/dependency/* org.example.BBAppStarter
   ```
4. Accede desde el navegador con la IP pública de tu instancia:
   ```
   http://ec2-54-227-116-251.compute-1.amazonaws.com:8080/
   ```

---

## 🎥 Evidencia
El video con la demostración funcional del proyecto se encuentra en la carpeta:  
📂 `Assets/EC2 Instance Connect _ us-east-1 - Brave 2025-10-29 19-01-11.mp4`
