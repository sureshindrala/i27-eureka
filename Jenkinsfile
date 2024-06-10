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
        SONAR_URL = "http://34.66.190.70:9000/"
        SONAR_TOKEN = credentials('sonar_creds')

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

          stage("build & SonarQube analysis") {
            steps {
              withSonarQubeEnv('SonarQube') {
               sh """
                echo "Starting Sonarqube"
                mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=i27-eureka \
                    -Dsonar.host.url=${env.SONAR_URL} \
                    -Dsonar.login=${env.SONAR_TOKEN}
                """
              }
            
          
            timeout(time: 5, unit: 'MINUTES') {
              script {
                waitForQualityGate abortPipeline: true

                }
              
              }
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
                docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}              
              
               """
            }
        }
        stage ('Deploy to Dev') {
          steps {
            echo "*********************Deploying to Devenvironment*************"
            withCredentials([usernamePassword(credentialsId: 'docker_env_creds', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
             //sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} hostname -i"

            // echo "***********pull image from the docker*****************"
            // sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
           // creating container
           //  sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker run -d -p 5761:8761 ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
          script {
            echo "*********** Pull The Image ***************************"
            sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
            //sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker pull ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"
             try {
              // stop the container
              echo ">>>>>>>>>>>>>>>>> stop the container <<<<<<<<<<<<<"
              sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker stop ${env.APPLICATION_NAME}-dev"
              //sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker stop ${env.APPLICATION_NAME}-dev"
              echo ">>>>>>>>>>>>>>>>> remove the container <<<<<<<<<<<<<<<<<<<<<<<<"
              sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker rm ${env.APPLICATION_NAME}-dev"
              //sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker rm ${env.APPLICATION_NAME}-dev"
 
            } catch (err) {
              echo "Caught the error : $err"
            }
              echo " ************** Creating the container *******************"
              sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} docker run -d -p 5761:8761 ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"     
            
          }
          
          }

        }
    }
  }
}





//sh "sshpass -p ${PASSWORD} -v ssh -o StrictHostKeyChecking=no ${USERNAME}@${docker_server_ip} hostname -i" 
// /home/sureshindrala1/jenkins/workspace/eureka_master/target/i27-eureka-0.0.1-SNAPSHOT.jar
// cp /home/i27k8s10/jenkins/workspace/i27-Eureka_master/target/i27-eureka-0.0.1-SNAPSHOT.jar ./.cicd

// cp ${workspace}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd
//  docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}

///
//steps {
  //            withSonarQubeEnv('My SonarQube Server') {
    //            sh 'mvn clean package sonar:sonar'
      //        }
        //    }
          //}
          //stage("Quality Gate") {
            //steps {
              //timeout(time: 1, unit: 'HOURS') {
                //waitForQualityGate abortPipeline: true
              //}
    //        /} 



  /*
    mvn clean verify sonar:sonar \
  -Dsonar.projectKey=i27-eureka \
  -Dsonar.host.url=http://34.66.190.70:9000 \
  -Dsonar.login=sqp_888f323cb8e0ba863de1055244da41b3d7c11300

// dev ==> 5761 (HP)
// test ==> 6761 (HP)
// stage ==> 7761 (HP)
// Prod ==> 8761 (HP)


  */