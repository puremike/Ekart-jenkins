pipeline {
    agent any
    tools {
        jdk "jdk" //uses the jdk configured tool name in the manage-jenkins/tools settings
        maven "maven" //uses the maven configured tool name in the manage-jenkins/tools settings 
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner' //uses the sonarqube configured tool name in the manage-jenkins/tools settings
    }

    stages {
        stage ("Git Checkout") {
            steps {
                git branch: 'main', changelog: false, credentialsId: 'eda9997a-e4a4-47e8-9238-c00d9edfd7fa', poll: false, url: 'https://github.com/puremike/Ekart-jenkins.git'
            }
        }
        
        stage ("Compile") {
            // It's neccessary to compile the application during development/testing phase rather than to package it. You can package it during the BUILD PHASE
            steps {
                sh "mvn clean compile -DskipTests=true"
            }
        }
        
        stage ("OWASP DP-Check") {
            steps {
                dependencyCheck additionalArguments: "--scan ./ --format HTML", odcInstallation: "dp"
                dependencyCheckPublisher pattern: "**/dependency-check-report.xml" 
            }
        }
        
        stage ("Sonar Config") {
            steps {
                // uses the sonarqube installation name in the manage-jenkins/global system settings.
                withSonarQubeEnv('sonar-server') {
                    /* sonar.projectKey: A unique identifier for the project (mandatory).
                    sonar.projectName: A readable name for the project (optional).
                    sonar.sources: Points to the source directory (use . if source files are in the root directory).
                    sonar.java.binaries: Points to the compiled .class files (usually in target/classes for Maven projects).
                    */
                    sh """ ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=Shopping-Cart \
                        -Dsonar.projectName=Shopping-Cart \
                        -Dsonar.sources=. \
                        -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }
        stage ("mvn-build") {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage ("docker-build and push") {
            steps {
                withDockerRegistry(credentialsId: '45a2af58-4171-482a-adff-2e7834d47008', url:"") {
                    sh "docker build -t scophee/shopping-cart-ekart:1.0 -f docker/Dockerfile ."
                    sh "docker push scophee/shopping-cart-ekart:1.0"
                }
            }
        }
        
         stage ("docker deploy") {
            steps {
                withDockerRegistry(credentialsId: '45a2af58-4171-482a-adff-2e7834d47008', url:"") {
                    sh "docker run -d --name shoping-c -p 8070:8070 scophee/shopping-cart-ekart:1.0"
                }
            }
        }
    }
}
