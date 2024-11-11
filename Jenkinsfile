pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git url: 'git@github.com:dsiyarp/devops-jenkins.git', credentialsId: '9ee12600-5468-467d-af03-ccd69b3a2bf4', branch: 'main'
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
