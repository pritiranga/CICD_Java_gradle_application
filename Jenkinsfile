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
                sshagent(['dev']) {                        
                    sh ''' 
                        ssh -o StrictHostKeyChecking=no testing@192.168.6.99 <<EOF 
                        echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin 
                        docker push pritidevops/k8-gradleapp:latest 
                        docker rmi -f k8-gradleapp:latest 
                        docker rmi -f pritidevops/k8-gradleapp:latest
EOF
                    '''
                }
            }
        }

        stage('Deploying application on k8s cluster') {
            steps {
                script {
                    sshagent(['k8']){
                            sh """
                                ssh -o StrictHostKeyChecking=no devsecops1@192.168.6.77 <<EOF
                                cd CICD_Java_gradle_application/kubernetes
                                helm package myapp
                                helm install priti myapp-0.3.0.tgz                                
                      """
                        
                    }
                }
            }
        }


        
        
    }
}

