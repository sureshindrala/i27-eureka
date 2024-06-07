pipeline {
    agent {
        label "k8s-slave"
    }
    environment {
        APPLICATION_NAME = "eureka"
        POM_VERSION = readMavenPom().getVersion()
        POM_PACKAGING = readMavenPom().getPackaging()
        // version + packaging
        DOCKER_HUB = "docker.io/sureshindrala"
        DOCKER_CREDS = credentials("dockerhub_creds")
        SONAR_URL = "http://34.66.190.70:9000 "
        SONAR_TOKEN = credentials("sonar_creds")
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
                echo "UnitTest for ${env.APPLICATION_NAME} Apllication"
                sh "mvn test"
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
                echo "Starting Sonar Scan"
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=${env.SONAR_URL} \
                    -Dsonar.login=${SONAR_TOKEN}
                '''
            }
        }
        stage ('Docker format') {
            steps {
                echo "JAR Source: ${env.APPLICATION_NAME}-${env.POM_VERSION}-${env.POM_PACKAGING}"  
            // need to have below farmating
            // eureka build number and formating 
            // eureka-06-master.jar
            echo "Jar Dest: ${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}-${env.POM_PACKAGING}"

            }
        }
        stage ('Docker Build') {
            steps{
                sh """
                ls -la
                cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
                                
                 ls -la ./.cicd
                 echo "***********Build Docker Image *******************"
                 docker build --force-rm --no-cache --pull --rm=true --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd
                 docker images
                 
                echo "************ Docker login *******************"
                docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}  

                echo "************** Docker Push ******************"
                docker push  ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}              
                

                """
            }
        }
    }
}
// /home/sureshindrala1/jenkins/workspace/eureka_master/target/i27-eureka-0.0.1-SNAPSHOT.jar
// cp /home/i27k8s10/jenkins/workspace/i27-Eureka_master/target/i27-eureka-0.0.1-SNAPSHOT.jar ./.cicd

// cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
//  docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}