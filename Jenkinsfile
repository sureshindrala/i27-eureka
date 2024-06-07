pipeline {
    agent {
        label 'k8s-slave'
    }
    environment {
        APPLICATION_NAME = 'eureka'
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        DOCKER_HUB = 'docker.io/sureshindrala'
        DOCKER_CREDS = credentials('dockerhub_creds')
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'Jdk-17'
    }

    stages {
        stage('build') {
            steps {
                echo "Building ${env.APPLICATION_NAME} application"
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Test') {
            steps {
                echo "Running unit tests for ${env.APPLICATION_NAME} application"
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        } 
        stage('sonar-test') {
            steps {
                sh '''
                echo "******** Sonar starting *************"
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=http://34.66.190.70:9000 \
                    -Dsonar.login=squ_401fe3557766b5b02aad46db890b0f7639d4cd58
                '''
            }
        }
        stage('Docker format') {
            steps {
                echo "JAR Source: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"
                echo "JAR Dest: ${env.APPLICATION_NAME}-${currentBuild.number}-${env.BRANCH_NAME}-${env.POM_PACKAGING}"
            }
        }
        stage('Docker Build') {
            steps {
                sh '''
                ls -la
                cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                
                ls -la ./.cicd
                echo "*********** Build Docker Image *******************"
                docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                docker images
                
                echo "************ Docker login *******************"
                docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                
                echo "************** Docker Push ******************"
                docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}
                '''
            }
        }
    }
}

// /home/sureshindrala1/jenkins/workspace/eureka_master/target/i27-eureka-0.0.1-SNAPSHOT.jar
// cp /home/i27k8s10/jenkins/workspace/i27-Eureka_master/target/i27-eureka-0.0.1-SNAPSHOT.jar ./.cicd

// cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
//  docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}