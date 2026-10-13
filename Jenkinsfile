pipeline {
    agent any

    stages {
        stage('Check Environment') {
            steps {
                sh 'echo "Jenkins pipeline is running"'
                sh 'docker --version'
                sh 'git --version'
            }
        }
    }
}
