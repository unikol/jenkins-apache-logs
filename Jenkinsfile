pipeline {
    agent any

    parameters {
        string(
            name: 'TARGET_HOST',
            defaultValue: '192.168.56.102',
            description: 'Target Ubuntu VM IP address'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Repository was checked out successfully.'
            }
        }

        stage('Show target host') {
            steps {
                echo "Target host is: ${params.TARGET_HOST}"
            }
        }

        stage('Test SSH connection from Jenkins') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'jenkins-apache-ssh-key',
                        keyFileVariable: 'SSH_KEY_FILE',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    bat """
                        echo Testing SSH connection from Jenkins to Linux VM...
                        echo Target host: ${params.TARGET_HOST}
                        echo SSH user: %SSH_USER%

                        ssh -i "%SSH_KEY_FILE%" -o StrictHostKeyChecking=no -o UserKnownHostsFile=NUL %SSH_USER%@${params.TARGET_HOST} "hostname && whoami && sudo -n whoami"
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'SSH connection from Jenkins to VM works successfully.'
        }

        failure {
            echo 'SSH connection test failed. Check Jenkins credentials, VM IP, SSH service, or sudo permissions.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}