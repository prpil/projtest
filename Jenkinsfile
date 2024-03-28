pipeline {
    agent any
    environment {
        DOCKERHUB_CREDENTIALS = credentials('docker_c')
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
                echo 'Testing..'
                withCredentials([usernamePassword(credentialsId: 'docker_cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        
                        docker tag prajipil/mydock:${env.BUILD_ID} prajipil/mydock:latest
                        docker push prajipil/mydock:latest
                    """
                }
            }
        }
    }
<<<<<<< HEAD
}
=======
}
>>>>>>> be559efc606b38132d200b4b9655e5976c4cc22f