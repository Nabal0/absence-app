pipeline {
    agent any

    environment {
        SSH_KEY = credentials('SSH-key-Jenkins')
        INVENTORY_PATH = 'ansible/inventory'
        PLAYBOOK_PATH = 'ansible/playbook.yml'
    }

    stages {
        stage('Checkout code') {
            steps {
                git branch: 'main', url: 'https://github.com/Nabal0/absence-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Build Docker image') {
            steps {
                sh 'docker build -t absence-app:latest .'
            }
        }

        stage('Run tests with Selenium') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Deploy to VM with Ansible') {
            steps {
                // Ajout de la clé privée dans un fichier temporaire
                sh '''
                    echo "$SSH_KEY" > /tmp/id_rsa
                    chmod 600 /tmp/id_rsa
                '''
                // Lancer le playbook
                sh '''
                    ansible-playbook -i ansible/inventory ansible/playbook.yaml --private-key /tmp/id_rsa --user nabil
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
