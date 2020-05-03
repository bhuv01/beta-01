pipeline {
    agent {label 'worker02'}
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t manneykapoor/beta-01:${DOCKER_TAG}"
            }
        }
        stage('Docker HUB Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u manneykapoor -p ${dockerHubPwd}"
                    sh "docker push manneykapoor/beta-01:${DOCKER_TAG}"
                }
            }
        }
        stage('Deploy to K8s'){
            steps{
				sh "chmod +x changeTag.sh"
				sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kops-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml app-pod.yml ubuntu@99.0.7.168:/home/ubuntu/"
					script{
						try{
							sh " ssh ubuntu@99.0.7.168 sudo kubectl apply -f ."
						}
						catch(error){
							sh "ssh ubuntu@99.0.7.168 sudo kubectl create -f ."
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
