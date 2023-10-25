pipeline{
    agent any

    environment {
        dockerHubRegistry = 'digitaltulbo'
        dockerHubRegistryCredential = 'docker-hub'
        githubCredential = 'github_cred'
        gitEmail = 'djzepssa@gmail.com'
        gitName = 'digitaltulbo'
        imageFrontend = 'frontend'
        imageBackend = 'backend'
        imageMysql= 'mysql'
    }

    stages {
        stage('check out application git branch'){
            steps {
                checkout scm
            }
            post {
                failure {
                    echo 'repository checkout failure'
                }
                success {
                    echo 'repository checkout success'
                }
            }
        }

        stage('docker image build'){
            steps{
                sh "docker build -t ${dockerHubRegistry}/${imageFrontend}:${currentBuild.number} ./frontend"
                sh "docker build -t ${dockerHubRegistry}/${imageFrontend}:latest ./frontend"

                sh "docker build -t ${dockerHubRegistry}/${imageBackend}:${currentBuild.number} ./backend"
                sh "docker build -t ${dockerHubRegistry}/${imageBackend}:latest ./backend"

                sh "docker build -t ${dockerHubRegistry}/${imageMysql}:${currentBuild.number} ./mysql"
                sh "docker build -t ${dockerHubRegistry}/${imageMysql}:latest ./mysql"
            }
            post {
                    failure {
                      echo 'Docker image build failure !'
                    }
                    success {
                      echo 'Docker image build success !'
                    }
            }
        }
        
        stage('Docker Image Push') {
            steps {
                withDockerRegistry([ credentialsId: dockerHubRegistryCredential, url: "" ]) {
                    sh "docker push ${dockerHubRegistry}/${imageFrontend}:${currentBuild.number}"
                    sh "docker push ${dockerHubRegistry}/${imageFrontend}:latest"

                    sh "docker push ${dockerHubRegistry}/${imageBackend}:${currentBuild.number}"
                    sh "docker push ${dockerHubRegistry}/${imageBackend}:latest"

                    sh "docker push ${dockerHubRegistry}/${imageMysql}:${currentBuild.number}"
                    sh "docker push ${dockerHubRegistry}/${imageMysql}:latest"

                    sleep 10 /* Wait uploading */
                }
            }
            post {
                    failure {
                      echo 'Docker Image Push failure !'
                      sh "docker rmi ${dockerHubRegistry}/${imageFrontend}:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/${imageFrontend}:latest"

                      sh "docker rmi ${dockerHubRegistry}/${imageBackend}:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/${imageBackend}:latest"

                      sh "docker rmi ${dockerHubRegistry}/${imageMysql}:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/${imageMysql}:latest"
                    }
                    success {
                      echo 'Docker image push success !'
                      sh "docker rmi ${dockerHubRegistry}/${imageFrontend}:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/${imageFrontend}:latest"

                      sh "docker rmi ${dockerHubRegistry}/${imageBackend}:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/${imageBackend}:latest"

                      sh "docker rmi ${dockerHubRegistry}/${imageMysql}:${currentBuild.number}"
                      sh "docker rmi ${dockerHubRegistry}/${imageMysql}:latest"
                    }
            }
        }
        stage('K8S Manifest Update') {
            steps {
                sh "ls"
                sh 'mkdir -p gitOpsRepo'
                dir("gitOpsRepo")
                {
                    git branch: "main",
                    credentialsId: githubCredential,
                    url: 'https://github.com/digitaltulbo/ingress-project.git'

                    sh "git config --global user.email ${gitEmail}"
                    sh "git config --global user.name ${gitName}"

               
                    sh "sed -i 's/frontend:.*\$/frontend:${currentBuild.number}/g' web-project.yaml"
                    sh "sed -i 's/backend:.*\$/backend:${currentBuild.number}/g' api-project.yaml"
                    sh "sed -i 's/mysql:.*\$/mysql:${currentBuild.number}/g' mysql-deployment.yaml"

                    sh "git add ."
                    sh "git commit -m '[UPDATE] k8s ${currentBuild.number} image versioning'"



                    withCredentials([gitUsernamePassword(credentialsId: githubCredential,
                                     gitToolName: 'git-tool')]) {
                        sh "git remote set-url origin https://github.com/digitaltulbo/ingress-project.git"
                        sh "git push -u origin main"
                    }
                }
            }
            post {
                    failure {
                      echo 'K8S Manifest Update failure !'
                    }
                    success {
                      echo 'K8S Manifest Update success !'
                    }
            }
        }

    }
}
