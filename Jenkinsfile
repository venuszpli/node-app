pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Checkout Source'){
            steps{
                git "https://github.com/venuszpli/node-app.git"
            }
        }
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t venuszpli/myweb:${DOCKER_TAG} "
            }
        }
        stage('DockHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u venuszpli -p ${dockerHubPwd}"
                    sh "docker push venuszpli/myweb:${DOCKER_TAG}"
                }
            }
        }
        stage('Docker Deploy k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops']) {
                    script{
                        try{
                            sh "ssh ec2-user@10.121.44.17 kubectl apply -f https://github.com/javahometech/node-app/blob/master/services.yml"
                            sh "ssh ec2-user@10.121.44.17 kubectl apply -f https://github.com/javahometech/node-app/blob/master/pods.yml"
                        }catch(error){
                            sh "ssh ec2-user@10.121.44.17 kubectl create -f https://github.com/javahometech/node-app/blob/master/services.yml"
                            sh "ssh ec2-user@10.121.44.17 kubectl create -f https://github.com/javahometech/node-app/blob/master/pods.yml"
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
