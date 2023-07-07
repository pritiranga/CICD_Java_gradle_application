pipeline{
    agent any 
    environment{
        DOCKERHUB_CREDENTIALS = credentials('Dockerhub')
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
                            ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null testing@192.168.6.99 << EOF
                            export PATH=$PATH:/opt/gradle/gradle-7.1.1/bin
                            cd /home/testing/CICD_Java_gradle_application
                            docker build -t k8-gradleapp:latest .
                            docker tag k8-gradleapp:latest pritidevops/k8-gradleapp:latest
EOF
                        '''
                    }    
                }
            }



        stage('Publishing Images to Dockerhub') {
            steps {
                echo "Pushing the image created to Dockerhub..."
                withCredentials([usernamePassword(credentialsId: 'DOCKERHUB_CREDENTIALS', usernameVariable: 'DOCKERHUB_USR', passwordVariable: 'DOCKERHUB_PSW')]){
                    sshagent(['dev']) {
                        sh 'ssh -o StrictHostKeyChecking=no testing@192.168.6.99 "echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin && docker push pritidevops/k8-gradleapp:latest && docker rmi -f k8-gradleapp:latest && docker rmi -f pritidevops/k8-gradleapp:latest"'
                    }
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
                    withCredentials([string(credentialsId: 'k8-config')]) {
                        sh "ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 \"cd CICD_Java_gradle_application/kubernetes && helm upgrade --install --set image.repository='pritidevops/k8-gradleapp:latest' gradlejavaapp myapp/\""
                    }
                }
            }
        }
        
        
    }
}

