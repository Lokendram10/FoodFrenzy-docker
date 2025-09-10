pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokendradhote64/FoodFrenzy:latest'
        SONARQUBE_SERVER = tool("SonarQubeServer")
    }

    stages {
        stage('Clone') {
            steps {
                git url: 'https://github.com/Lokendram10/FoodFrenzy-docker', branch: 'master'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                  sh "${SONARQUBE_SERVER}/bin/sonar-scanner   -DsonarProjectName=FoodFrenzy -Dsonar.projectKey=FoodFrenzy "

                }
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy fs  --severity HIGH,CRITICAL -f html -o trivy-report.html .'
            }
        }
        stage('Owasp Dependency Check'){
                 steps {
                     withSonarQubeEnv('SonarQubeServer') {
                    sh ''' ${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=FoodFrenzy \
                     -Dsonar.projectKey=FoodFrenzy
                     '''
                     }
                 }
            }
             stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        

        stage('Build Docker Image') {
              steps {
               echo 'Building Docker Image'
                sh "docker build -t ${DOCKER_IMAGE} ."
               }
      }  
        stage('Push to  Docker HUb'){
           steps{
             echo'Pushing Image to Docker Hub'
                 withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                 passwordVariable: 'DOCKERHUB_PASSWORD', 
                 usernameVariable: 'DOCKERHUB_USERNAME')]) {
              sh 'docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD'
              sh 'docker push ${DOCKER_IMAGE}'
            }
          }
         }
        stage('Docker Compose Up') {
            steps {
                sh 'docker-compose up -d --build'
            }
        }
    }
    post {
        always {
            sh 'docker-compose '
        }
    }
}