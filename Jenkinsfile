pipeline {
    agent any

    environment {
        IMAGE_NAME = "ravikiiran/hello-world"
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_CREDENTIALS = "dockerhub-creds"

        KIND_CLUSTER = "myapp-cluster"
    }

    stages {

        stage('Build') {
            steps {
                sh '''
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.4.0.6343:sonar \
                            -Dsonar.projectKey=hello-world \
                            -Dsonar.projectName="Hello World"
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kind') {
            steps {
                sh '''
                    kubectl config use-context kind-${KIND_CLUSTER}

                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml

                    kubectl set image deployment/hello-world \
                        hello-world=${IMAGE_NAME}:${IMAGE_TAG}

                    kubectl rollout status deployment/hello-world \
                        --timeout=120s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "===== PODS ====="
                    kubectl get pods -o wide

                    echo "===== DEPLOYMENT ====="
                    kubectl get deployment hello-world

                    echo "===== SERVICE ====="
                    kubectl get service hello-world

                    echo "===== IMAGE ====="
                    kubectl get deployment hello-world \
                        -o jsonpath='{.spec.template.spec.containers[0].image}'

                    echo
                '''
            }
        }
    }
}
