Spring Boot Shopping Cart Web App
=================================

Overview
--------

This project demonstrates how to set up and run a Continuous Integration and Continuous Deployment (CI/CD) pipeline using **Jenkins**, leveraging a pre-existing Spring Boot Shopping Cart Web App codebase. The application features a login and registration system, shopping cart functionality with session persistence, and a transactional checkout process.

The web app is built using:

-   **Spring Boot**

-   **Spring Security**

-   **Thymeleaf**

-   **Spring Data JPA**

-   **Spring Data REST**

-   **Docker**

The backend is powered by an in-memory **H2 Database**.

* * * * *

Configuration
-------------

### Configuration Files

The configuration files for the application are located in the **src/resources/** directory:

-   **application.properties**: The main configuration file. You can modify admin credentials and port number here.

* * * * *

CI/CD Pipeline Using Jenkins
----------------------------

### Jenkins Setup

1.  **Install Jenkins**:

    -   Set up Jenkins on a local Ubuntu system.

2.  **Install Necessary Plugins**:

    -   Plugins installed include:

        -   SonarQube

        -   Docker (Docker Commons, Docker API plugin, Docker Pipeline, Docker Build Step, CloudBees Docker Build and Publish Plugin)

        -   OWASP Dependency Check Plugin

        -   JRE (Eclipse Temurin installer Plugin)

3.  **Configure Tools and Credentials**:

    -   Configured tools (e.g., Maven, SonarQube, Docker).

    -   Set up credentials for GitHub, Docker Hub, SonarQube, and others.

### Pipeline Stages

#### 1\. Git Checkout

Clones the repository from GitHub:

```
stage ("Git Checkout") {
    steps {
        git branch: 'main', changelog: false, credentialsId: 'eda9997a-e4a4-47e8-9238-c00d9edfd7fa', poll: false, url: 'https://github.com/puremike/Ekart-jenkins.git'
    }
}
```

#### 2\. Compile Code

Compiles the application without running tests:

```
stage ("Compile") {
    steps {
        sh "mvn clean compile -DskipTests=true"
    }
}
```

#### 3\. OWASP Dependency Check

Performs a security check for dependencies:

```
stage ("OWASP DP-Check") {
    steps {
        dependencyCheck additionalArguments: "--scan ./ --format HTML", odcInstallation: "dp"
        dependencyCheckPublisher pattern: "**/dependency-check-report.xml"
    }
}
```

#### 4\. SonarQube Configuration

Configures and runs SonarQube for code analysis:

1.  **Setup**:

    -   Pulled the SonarQube Docker image and ran it locally:

        ```
        docker run --name sonar -p 9000:9000 sonarqube:lts-community
        ```

    -   Logged into `localhost:9000` with default credentials and generated an API token.

2.  **Pipeline**:

    ```
    stage ("Sonar Config") {
        steps {
            withSonarQubeEnv('sonar-server') {
                sh """ ${SCANNER_HOME}/bin/sonar-scanner\
                    -Dsonar.projectKey=Shopping-Cart\
                    -Dsonar.projectName=Shopping-Cart\
                    -Dsonar.sources=.\
                    -Dsonar.java.binaries=target/classes
                """
            }
        }
    }
    ```

#### 5\. Maven Build

Packages the application:

```
stage ("mvn-build") {
    steps {
        sh "mvn clean package -DskipTests=true"
    }
}
```

#### 6\. Docker Build and Push

Builds a Docker image and pushes it to Docker Hub:

```
stage ("docker-build") {
    steps {
        withDockerRegistry(credentialsId: '45a2af58-4171-482a-adff-2e7834d47008', url:"") {
            sh "docker build -t scophee/shopping-cart-ekart:1.0 -f docker/Dockerfile ."
            sh "docker push scophee/shopping-cart-ekart:1.0"
        }
    }
}
```

#### 7\. Docker Deployment

Deploys the application using Docker:

```
stage ("docker deploy") {
    steps {
        withDockerRegistry(credentialsId: '45a2af58-4171-482a-adff-2e7834d47008', url:"") {
            sh "docker run -d --name shoping-c -p 8070:8070 scophee/shopping-cart-ekart:1.0"
        }
    }
}
```

### Complete Pipeline Script

```
pipeline {
    agent any
    tools {
        jdk "jdk" // Configured JDK tool name in Jenkins
        maven "maven" // Configured Maven tool name in Jenkins
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner' // Configured SonarQube tool name in Jenkins
    }
    stages {
        stage ("Git Checkout") { ... }
        stage ("Compile") { ... }
        stage ("OWASP DP-Check") { ... }
        stage ("Sonar Config") { ... }
        stage ("mvn-build") { ... }
        stage ("docker-build") { ... }
        stage ("docker deploy") { ... }
    }
}
```

* * * * *

How to Run Locally
------------------

### Maven

To run the application locally, use the Maven Wrapper or Maven CLI:

#### Using the Maven Plugin

```
$ mvn spring-boot:run
```

#### Building and Running the JAR File

```
$ mvn clean package
$ java -jar target/shopping-cart-0.0.1-SNAPSHOT.jar
```

### Docker

Build and run the application using Docker:

```
$ mvn clean package
$ docker build -t shopping-cart:dev -f docker/Dockerfile .
$ docker run --rm -i -p 8070:8070 --name shopping-cart shopping-cart:dev
```

* * * * *

Additional Resources
--------------------

### H2 Database Console

Access the database console at `http://localhost:8070/h2-console` and use the following credentials:

-   **JDBC URL**: `jdbc:h2:mem:shopping_cart_db`

### HAL REST Browser

Access the REST API browser at `http://localhost:8070/`.

* * * * *

Testing
-------

Run tests using Maven:

```
$ mvn test
```
