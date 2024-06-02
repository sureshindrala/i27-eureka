pipeline {
    agent {
        label "k8s-slave"
    }
    environment {
        APPLICATION_NAME = "eureka"
        
    }
    tools{
        maven 'Maven-3.8.8'
        jdk 'Jdk-17'
    }

    stages {
        stage ('build') {
            // building the application
            steps{
                echo "build  ${env.APPLICATION_NAME} Application "
                sh "mvn clean package -DskipTests=true"
            }

        }
        stage('Unit Test') {
            steps{
                echo "UnitTest for ${env.APPLICATION_NAME} Apllicatio"
                sh "mvn test"
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

    }


}