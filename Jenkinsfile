pipeline {
    agent any

    environment {
        SSH_KEY = credentials('SSH-key-Jenkins')        // Clé privée pour Ansible
        DOCKERHUB_USER = credentials('nabilbelz')       // ID Jenkins pour DockerHub
        IMAGE_NAME = 'nabal0/absence-app'
        INVENTORY_PATH = 'ansible/inventory'
        PLAYBOOK_PATH = 'ansible/playbook.yaml'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'git@github.com:Nabal0/absence-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install -DskipTests=true'
            }
        }

        stage('Run Selenium Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:latest .'
            }
        }

        stage('Push Docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-user', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                sh '''
                    echo "$SSH_KEY" > /tmp/id_rsa
                    chmod 600 /tmp/id_rsa
                    ansible-playbook -i $INVENTORY_PATH $PLAYBOOK_PATH --private-key /tmp/id_rsa --user nabil
                '''
            }
        }
    }

    post {
        always {
            sh 'rm -f /tmp/id_rsa'
        }
    }
}
