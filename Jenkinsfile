pipeline {
    agent any
    environment {
        IMAGE_NAME = "meghakimshuka/docker-react-app"
        IMAGE_TAG = "${currentBuild.number}"
        GIT_REPO = "https://github.com/Meghana0976/AWS-EC2-Docker-CICD"
        GIT_BRANCH = "main"
        EC2_USER = "ubuntu"
        EC2_IP = "3.87.57.12"
        // Remove the direct path to the PEM key and use Jenkins credentials
        SSH_KEY = "ocker-cicd-ssh-key" // Reference to Jenkins Secret File (SSH key)
    }
    stages {
        stage("Git Checkout") {
            steps {
                script {
                    git url: "${GIT_REPO}", branch: "${GIT_BRANCH}"
                }
            }
        }
        stage("Docker Build") {
            steps {
                script {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        stage("Docker Push") {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DockerHub', passwordVariable: 'docker_password', usernameVariable: 'docker_user')]) {
                    script {
                        sh "echo \"${docker_password}\" | docker login -u \"${docker_user}\" --password-stdin"
                    }
                }
                script {
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
            }
        }

        stage("Dev Deploy") {
            steps {
                // Use the sshagent plugin to load the SSH key stored in Jenkins credentials
                sshagent(['ocker-cicd-ssh-key']) {
                    script {
                        // Ensure you're using the correct user for EC2 (ubuntu for Ubuntu-based EC2)
                        sh "ssh -o StrictHostKeyChecking=no ubuntu@${EC2_IP} 'sudo docker rm -f react 2>/dev/null || true'"
                        sh "ssh ubuntu@${EC2_IP} 'sudo docker run -d -p 3000:3000 --name final-deploy-container ${IMAGE_NAME}:${IMAGE_TAG}'"
                    }
                }
            }
        }
    }
}
