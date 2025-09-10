pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'lokendradhote64/FoodFrenzy:latest'
        SONARQUBE_SERVER = ("SonarQubeServer")
    }

    stages {
        stage('Clone') {
            steps {
                git url: 'https://github.com/Lokendram10/FoodFrenzy-docker', branch: 'master'
            }
        }
 
        stage('Trivy Scan') {
            steps {
                sh "trivy fs  --severity HIGH,CRITICAL -f html -o trivy-report.html ."
                archiveArtifacts artifacts: 'trivy-report.html'
            }
        }
        stage('SonarQube Scan') {
                 steps {
                     withSonarQubeEnv('SonarQubeServer') {
                    sh """ ${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner \
                    -Dsonar.projectName=FoodFrenzy \
                     -Dsonar.projectKey=FoodFrenzy \
                     -Dsonar.sources=.
                     """
                     }
                 }
            }
        stage('Owasp Dependency Check'){
                 steps {
                  echo "OWASP Dependency Check"
                     dependencyCheck(
                     additionalArguments: '--scan ./ --format HTML ',
                     odcInstallation: 'owasp',
                    )
                     dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                     archiveArtifacts artifacts: 'dependency-check-report.html'
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
              sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
              sh "docker push ${DOCKER_IMAGE}"
            }
          }
         }
        stage('Deploy with Docker Compose') {
            steps {
                echo 'Deploying Application + MySQL using Docker Compose'
                sh "docker-compose pull && docker-compose up -d"
            }
        }
    }
   
}