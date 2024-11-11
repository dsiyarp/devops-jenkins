pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/dsiyarp/devops-jenkins.git'
            }
        }
        stage('Build') {
            steps {
                echo 'Compilando el proyecto...'
            }
        }
        stage('Deploy') {
            steps {
                sh 'scp -r . davidvm2@192.168.0.156:/var/www/html/'
            }
        }
    }
}
