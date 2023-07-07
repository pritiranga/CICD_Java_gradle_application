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
                    sh '''
                        ssh -o StrictHostKeyChecking=no testing@192.168.6.99 << EOF
                        export PATH=$PATH:/opt/gradle/gradle-7.1.1/bin
                        cd /home/testing/CICD_Java_gradle_application
                        docker build -t k8-gradleapp:${VERSION} .
                        docker tag k8-gradleapp:${VERSION} pritidevops/k8-gradleapp:${VERSION}
EOF
                    '''
                }
            }
        }


        stage('Publishing Images to Dockerhub') {
            steps {
                echo "Pushing the image created to Dockerhub..."
                sshagent(['dev']) {
                    sh 'ssh -o StrictHostKeyChecking=no testing@192.168.6.99 "echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin && docker push pritidevops/k8-gradleapp:${VERSION} && docker rmi -f k8-gradleapp:${VERSION} && docker rmi -f pritidevops/k8-gradleapp:${VERSION}"'
                }
            }
        }



        stage('manual approval'){
            steps{              
                script {
                    timeout(time: 10, unit: 'MINUTES'){
                        input ('Deploy to Production?')
                    }
                } 

            }
        }
        
        
        stage('Deploying application on k8s cluster') {
            steps {
                script {
                    withCredentials([kubeconfigFile(credentialsId: 'k8-config')]) {
                        sh "ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 \"cd CICD_Java_gradle_application/kubernetes && helm upgrade --install --set image.repository='pritidevops/k8-gradleapp' --set image.tag='${VERSION}' gradlejavaapp myapp/\""
                    }
                }
            }
        }
        
        
    }
}

