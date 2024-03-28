pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_c')
        EC2_PRIVATE_KEY = credentials('ec_id')
        EC2_INSTANCE_NAME = 'praj-dev' // Update with your EC2 instance name
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'python3 manage.py test'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("prajipil/mydock:${env.BUILD_ID}")
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_c', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "\$DOCKER_PASS" | docker login -u "\$DOCKER_USER" --password-stdin
                        docker tag prajipil/mydock:${env.BUILD_ID} prajipil/mydock:latest
                        docker push prajipil/mydock:latest
                    """
                }
            }
        }

        stage('Deploy to EC2 Instance') {
            steps {
                withCredentials([file(credentialsId: 'ec_id', variable: 'EC2_PRIVATE_KEY_FILE')]) {
                    sh """
                        chmod 600 \$EC2_PRIVATE_KEY_FILE
                        scp -i \$EC2_PRIVATE_KEY_FILE -o StrictHostKeyChecking=no docker-compose.yaml ec2-user@${EC2_INSTANCE_NAME}:/home/ec2-user/
                    """
                }
            }
        }

        stage('Run Selenium Tests') {
            steps {
                sshagent(['ec_id']) {
                    sh """
                        ssh -i \$EC2_PRIVATE_KEY_FILE ec2-user@${EC2_INSTANCE_NAME} 'docker-compose -f /home/ec2-user/docker-compose.yaml up -d'
                        python3 selenium.py
                    """
                }
            }
        }
    }
}
