pipeline {
    agent any

    tools {
        jdk 'JDK17'          // Jenkins JDK tool name (Java 17)
        maven 'maven'        // Jenkins Maven tool name
    }

    environment {
        DOCKER_IMAGE = 'lokendradhote64/FoodFrenzy:latest'
        SONARQUBE_SERVER = 'Sonar Cube Scanner'   // exact name of SonarQube Scanner tool
    }

    stages {

        // 1️⃣ Clone the repository
        stage('Clone') {
            steps {
                git url: 'https://github.com/Lokendram10/FoodFrenzy-docker', branch: 'master'
            }
        }

        // 2️⃣ Build Java Project
        stage('Build Java Project') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 3️⃣ Trivy Scan
        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL --format table -o trivy-report.txt .'
                sh 'trivy fs --severity HIGH,CRITICAL --format json -o trivy-report.json .'
                archiveArtifacts artifacts: 'trivy-report.*'
            }
        }

        // 4️⃣ OWASP Dependency Check
        stage('OWASP Dependency Check') {
            steps {
                echo "Running OWASP Dependency Check"
                dependencyCheck(
                    additionalArguments: '--scan ./',
                    odcInstallation: 'owasp'
                )
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                archiveArtifacts artifacts: 'dependency-check-report.html'
            }
        }

        // 5️⃣ SonarQube Scan
        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('Sonar Cube Scanner') {
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

        // 6️⃣ SonarQube Quality Gate
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        // 7️⃣ Build Docker Image
        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image'
                sh "docker build -t ${DOCKER_IMAGE} ."
            }
        }

        // 8️⃣ Push to Docker Hub
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

        // 9️⃣ Deploy using Docker Compose
        stage('Deploy') {
            steps {
                echo 'Deploying Application + MySQL using Docker Compose'
                sh 'docker-compose pull && docker-compose up -d'
            }
        }

    } // stages

    post {
        always {
            echo 'Pipeline finished. Archiving artifacts.'
            archiveArtifacts artifacts: '**/*.html, **/*.json'
        }
    }

}
