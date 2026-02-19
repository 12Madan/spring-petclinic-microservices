pipeline {
    agent any

    environment {
        DOCKER_REPO = "madanreddy9550"
        IMAGE_TAG = "${BUILD_NUMBER}" 
    }

    stages {

        stage('Workspace Clean') {
            steps {
                echo "Cleaning workspace..."
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/12Madan/spring-petclinic-microservices.git'
            }
        }

        stage('Java & Maven Version Check') {
            steps {
                sh 'java -version'
                sh 'mvn -version'
            }
        }

        stage('Maven Build (Multi-Module)') {
            steps {
                echo "Building all microservices..."
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=spring-petclinic-microservices \
                    -Dsonar.projectName=spring-petclinic-microservices
                    """
                }
            }
        }

        stage('Sonar Quality Gate') {
            steps {
                timeout(time: 5, unit: 'SECONDS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build & Push - All Microservices') {
            steps {
                script {

                    def services = [
                        "spring-petclinic-admin-server",
                        "spring-petclinic-api-gateway",
                        "spring-petclinic-config-server",
                        "spring-petclinic-discovery-server",
                        "spring-petclinic-customers-service",
                        "spring-petclinic-visits-service",
                        "spring-petclinic-vets-service",
                        "spring-petclinic-genai-service"
                    ]

                    withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {

                        sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        """

                        for (service in services) {

                            echo "Building and pushing ${service}:${IMAGE_TAG}"

                            sh """
                            docker build -t ${DOCKER_REPO}/${service}:${IMAGE_TAG} ./${service}
                            docker push ${DOCKER_REPO}/${service}:${IMAGE_TAG}

                            # Optional: also push latest
                            docker tag ${DOCKER_REPO}/${service}:${IMAGE_TAG} ${DOCKER_REPO}/${service}:latest
                            docker push ${DOCKER_REPO}/${service}:latest
                            """
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
        always {
            echo "Pipeline execution completed."
        }
    }
}
