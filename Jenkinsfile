pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                echo 'Repository was checked out successfully.'
            }
        }

        stage('Test Jenkins Pipeline') {
            steps {
                echo 'Jenkins is able to read Jenkinsfile from GitHub.'
                echo 'SoftServe Jenkins Apache logs task started.'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }
    }
}