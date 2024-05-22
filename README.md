### CI/CD: Deploy application to k8s in Windows OS with Jenkins Declarative CI/CD

1. Add k8s required plugins
   ![2](https://github.com/javatechie-devops/user-service/assets/25712816/247733e7-1ea4-4cc4-961e-49ceb3c267e2)
2. After adding the plugin in your Jenkins script, click on Generate pipeline syntax.
3. Next, from the dropdown, select kubeconfig
   ![3](https://github.com/javatechie-devops/user-service/assets/25712816/e5164c9a-fa93-4336-b82a-2451dbbd8007)
4. Next, in the credential section, click on add -> Jenkins
   ![4](https://github.com/javatechie-devops/user-service/assets/25712816/2dfb01a8-61d7-4938-8242-13f41de2062d)
5. Next, select secret file from the dropdown and provide the path of your kube config. Usually, you will find your kubeconfig file from the below path
   ![1](https://github.com/javatechie-devops/user-service/assets/25712816/b6361ae4-6ad2-4a81-83de-f650c29447ba)
   Once you execute the select command, then it will appear content like below
   ![9](https://github.com/javatechie-devops/user-service/assets/25712816/6ad56d68-398f-44fa-aba3-2936c5ff95bf)
   Just upload this file in the credential section like this
   ![5](https://github.com/javatechie-devops/user-service/assets/25712816/448ae917-0c4a-4758-899e-da5a10af096a)
7. Next, once you complete this, it will generate syntax for you
   ![6](https://github.com/javatechie-devops/user-service/assets/25712816/b203cae3-07af-4803-b521-f190c087fae7), copy this and use it in your script.
8. Use the below script for Windows:
   ```groovy
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
           DOCKER_USER = "javatechie4u"
           IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
           IMAGE_TAG = "${RELEASE_NO}-${BUILD_NUMBER}"
       }

       stages {
           stage("SCM checkout") {
               steps {
                   // Checkout code from Git
                   checkout([$class: 'GitSCM', branches: [[name: '*/main']], userRemoteConfigs: [[url: 'https://github.com/javatechie-devops/user-service.git']]])
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
                   // Login to Docker Hub
                    withCredentials([string(credentialsId: 'dpwd', variable: 'dpwd')]) {
                       bat "docker login -u ${DOCKER_USER} -p ${dpwd}"
                       bat "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                   }
               }
           }

           stage("Deploy to Kubernetes") {
               steps {
                   script {
                       kubeconfig(credentialsId: 'kubeconfig', serverUrl: '') {
                           bat """powershell -Command "(Get-Content k8s-app.yaml) -replace 'image: .*', 'image: ${IMAGE_NAME}:${IMAGE_TAG}' | Set-Content k8s-app.yaml" """
                           bat 'kubectl apply -f k8s-app.yaml'
                       }
                   }
               }
           }

           stage("Verify deployment") {
               steps {
                   script {
                       kubeconfig(credentialsId: 'kubeconfig', serverUrl: '') {
                           // Check deployed pods
                           bat 'kubectl get pods -n user-app'
                       }
                   }
               }
           }
       }
   }
8. Build your job then you could see successful results in the pipeline ![8](https://github.com/javatechie-devops/user-service/assets/25712816/63777278-918a-4486-b19f-0fe8e5a80f3a)

9. validate k8s componenet ![7](https://github.com/javatechie-devops/user-service/assets/25712816/810133d3-5c56-46e5-b677-275f1cd3864e)
