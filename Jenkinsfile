pipeline{
    agent any 
    environment{
        VERSION = "${env.BUILD_ID}"
        DOCKERHUB = credentials('Dockerhub')
    }
    
    tools {
        gradle 'Gradle'
    }
    
    stages{

        stage('Checkout') {
            steps {
                echo "Checkout the code from GitLab repo..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building the docker file..."
                sshagent(['dev']) {
                    sh 'ssh -o StrictHostKeyChecking=no testing@192.168.6.99 "export PATH=$PATH:/opt/gradle/gradle-7.1.1/bin && cd /home/testing/CICD_Java_gradle_application && docker build -t k8-gradleapp:${VERSION} . && docker tag k8-gradleapp:${VERSION} pritidevops/k8-gradleapp:${VERSION}"'
                }
            }
        }

        stage('Publishing Images to Dockerhub') {
            steps {
                echo "Pushing the image created to Dockerhub..."
                sshagent(['dev']) {
                    sh 'ssh -o StrictHostKeyChecking=no testing@192.168.6.99 "echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin && docker push pritidevops/k8-app:${VERSION} && docker rmi -f k8-app:${VERSION} && docker rmi -f pritidevops/k8-app:${VERSION}"'
                }
            }
        }



        stage('manual approval'){
            steps{
                script{
                    timeout(10) {
                        mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> Go to build url and approve the deployment request <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
                        input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
                    }
                }
            }
        }
        stage('Deploying application on k8s cluster') {
            steps {
               script{
                   withCredentials([kubeconfigFile(credentialsId: 'k8-config', variable: 'KUBECONFIG')]) {
			   sshagent(['k8-config']) {
                          	sh 'ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 "cd CICD_Java_gradle_application/kubernetes && helm upgrade --install --set image.repository="pritidevops/k8-gradleapp" --set image.tag="${VERSION}" gradlejavaapp myapp/ "' 
                        }
                  }
               }
            }
        }
        }
    }
<<<<<<< HEAD
=======

>>>>>>> f71fc4a91cd51be6d2523980ac955b68cd14f5c4
