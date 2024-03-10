// This Jenkinsfile is for the Eureka Deployment.
pipeline {
    agent {
        label 'k8s-slave'
    }
    environment {
        APPLICATION_NAME = "eureka"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        //version+ packaging
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
    stages {
        stage ('Build'){
            // Application Build happens here
            steps { // jenkins env variable no need of env 
                echo "Building the ${env.APPLICATION_NAME} application"
                sh "mvn clean package -DskipTests=true"
                //-DskipTests=true 
            }
        }
        stage ('Unit Tests') {
            steps {
                echo "Performing Unit tests for ${env.APPLICATION_NAME} application"
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        stage ('Docker Format') {
            steps {
                // Tell me, how can i read a pom.xml from jenkinfile
                echo "JAR Source: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
            }
        }
    }
}