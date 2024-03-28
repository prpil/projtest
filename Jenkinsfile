pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_c')
        EC2_PRIVATE_KEY = credentials('ec_id')
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
                script {
                    // Use AWS CLI to describe EC2 instances and extract the instance ID
                    def instanceId = sh(script: 'aws ec2 describe-instances --filters Name=tag:Name,Values=praj-dev --query "Reservations[*].Instances[*].InstanceId" --output text', returnStdout: true).trim()
                    
                    // Use AWS CLI to describe EC2 instances and extract the instance IP address
                    def instanceIp = sh(script: "aws ec2 describe-instances --instance-ids ${instanceId} --query 'Reservations[].Instances[].PublicIpAddress' --output text", returnStdout: true).trim()
                    
                    // Copy Docker Compose file to EC2 instance
                    sh """
                        scp -i /path/to/your/private/key.pem -o StrictHostKeyChecking=no docker-compose.yaml ec2-user@${instanceIp}:/home/ec2-user/
                    """
                }
            }
        }

        stage('Run Selenium Tests') {
            steps {
                // Run Selenium tests on the EC2 instance
                sshagent(['ec_id']) {
                    script {
                        // Use AWS CLI to describe EC2 instances and extract the instance ID
                        def instanceId = sh(script: 'aws ec2 describe-instances --filters Name=tag:Name,Values=praj-dev --query "Reservations[*].Instances[*].InstanceId" --output text', returnStdout: true).trim()
                        
                        // Use AWS CLI to describe EC2 instances and extract the instance IP address
                        def instanceIp = sh(script: "aws ec2 describe-instances --instance-ids ${instanceId} --query 'Reservations[].Instances[].PublicIpAddress' --output text", returnStdout: true).trim()
                        
                        // SSH into the EC2 instance and run Selenium tests
                        sh """
                            ssh -i /path/to/your/private/key.pem ec2-user@${instanceIp} 'docker-compose -f /home/ec2-user/docker-compose.yaml up -d'
                            python3 selenium_test.py
                        """
                    }
                }
            }
        }
    }
}
