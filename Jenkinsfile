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
                   withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                          sh 'helm upgrade --install --set image.repository="34.125.214.226:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
                        }
                    }
               }
            }
        }
        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                         sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'

                     }
                }
            }
        }
    }

    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "deekshith.snsep@gmail.com";  
		 }
	   }
}
