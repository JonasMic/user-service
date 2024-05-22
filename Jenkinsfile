pipeline {
    agent any

    tools {
        // Define Maven tool
        maven "maven"
    }

    environment {
        // Define environment variables
        APP_NAME = "user-service"
        RELEASE_NO = "1.0.0"
        DOCKER_USER = "cloud1592017"
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE_NO}-${BUILD_NUMBER}"
    }

    stages {
        stage("SCM checkout") {
            steps {
                // Checkout code from Git
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/JonasMic/user-service.git']]])
            }
        }

        stage("Build Process") {
            steps {
                // Build the project using Maven
                bat 'mvn clean install'
            }
        }

        stage("Build Image") {
            steps {
                // Build Docker image
                bat "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage("Deploy Image to Hub") {
            steps {
                // Login to Docker Hub and push the the specified image
                 withCredentials([string(credentialsId: 'dpwd', variable: 'dpwd')]) {
                    bat "docker login -u ${DOCKER_USER} -p ${dpwd}"
                    bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage("Deploy to Kubernetes") {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-pwd', serverUrl: '') {
                        bat """powershell -Command "(Get-Content k8s-app.yaml) -replace 'image: .*', 'image: ${IMAGE_NAME}:${IMAGE_TAG}' | Set-Content k8s-app.yaml" """
                        bat 'kubectl apply -f k8s-app.yaml'
                    }
                }
            }
        }

        stage("Verify deployment") {
            steps {
                script {
                    kubeconfig(credentialsId: 'kubeconfig-pwd', serverUrl: '') {
                        // Check deployed pods
                        bat 'kubectl get pods -n user-app'
                    }
                }
            }
        }
    }
}