pipeline {
    agent any

    tools {
        jdk 'JDK17'         // Make sure this matches Jenkins Global Tool name
        maven 'maven'        // Your Maven 3.9.11 installation name
    }

    environment {
        DOCKER_IMAGE = 'lokendradhote64/foodfrenzy:latest'
        SONARQUBE_SERVER = 'SonarqubeScanner'   // must match Jenkins SonarQube server name
    }

    stages {

        // Clone the repository
        stage('Clone') {
            steps {
                git url: 'https://github.com/Lokendram10/FoodFrenzy-docker', branch: 'master'
            }
        }

        //  Verify Tools
        stage('Verify Tools') {
            steps {
                sh 'java -version'
                sh 'mvn -v'
            }
        }

        //  Build Java Project
        stage('Build Java Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // Trivy Scan
        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --format table -o trivy-report.txt .'
                sh 'trivy fs --severity HIGH,CRITICAL --format json -o trivy-report.json .'
                archiveArtifacts artifacts: 'trivy-report.*'
            }
        }

        //  OWASP Dependency Check
  stage('OWASP Dependency Check') {
    steps {
        echo "Running OWASP Dependency Check"
        dependencyCheck(
            additionalArguments: '--scan ./ --format HTML',
            odcInstallation: 'owasp'
        )
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        archiveArtifacts artifacts: 'dependency-check-report.html, **/dependency-check-report.xml'
    }
}


        //  SonarQube Scan
        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_SERVER}") {
                    script {
                        def scannerHome = tool 'SonarqubeScanner'   // must match Jenkins tool name
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

        //  SonarQube Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // Build Docker Image
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        // Push to Docker Hub
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker Image'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'DOCKERHUB_USERNAME',
                                                 passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                    sh "docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                    sh "docker push ${DOCKER_IMAGE}"
                }
            }
        }

        //  Deploy using Docker Compose
        stage('Deploy') {
            steps {
                echo 'Deploying Application + MySQL using Docker Compose'
                sh 'docker compose pull && docker compose up -d'

            }
        }
    } 

    post {
        always {
            echo 'Pipeline finished. Archiving artifacts.'
            archiveArtifacts artifacts: '**/*.html, **/*.json'
        }
    }
}
