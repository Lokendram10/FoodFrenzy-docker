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
        // Human-readable scan (console + artifact)
        sh """
        trivy fs --severity HIGH,CRITICAL \
        --format table \
        -o trivy-report.txt .
        """

        // Machine-readable scan (JSON for tools)
        sh """
        trivy fs --severity HIGH,CRITICAL \
        --format json \
        -o trivy-report.json .
        """

        archiveArtifacts artifacts: 'trivy-report.*'
    }
}

     stage('Build Java Project') {
    steps {
        def mvnHome = tool 'maven'  // <-- exact name you configured
            sh "${mvnHome}/bin/mvn clean package -DskipTests"
    }
}

stage('SonarQube Scan') {
    steps {
        withSonarQubeEnv('SonarQubeServer') {
            script {
                def scannerHome = tool 'Sonar Cube Scanner'
                sh """
                ${scannerHome}/bin/sonar-scanner \
                  -Dsonar.projectKey=FoodFrenzy \
                  -Dsonar.projectName=FoodFrenzy \
                  -Dsonar.sources=. \
                  -Dsonar.java.binaries=target/classes
                """
            }
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