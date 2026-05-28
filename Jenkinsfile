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
                        echo Current Jenkins Windows user:
                        whoami

                        echo.
                        echo Testing SSH connection from Jenkins to Linux VM...
                        echo Target host: ${params.TARGET_HOST}
                        echo SSH user: %SSH_USER%
                        echo SSH key file: %SSH_KEY_FILE%

                        echo.
                        echo Fixing SSH private key permissions for Windows OpenSSH...

                        for /f "delims=" %%U in ('whoami') do set "CURRENT_USER=%%U"

                        icacls "%SSH_KEY_FILE%"
                        icacls "%SSH_KEY_FILE%" /inheritance:r
                        icacls "%SSH_KEY_FILE%" /grant:r "%CURRENT_USER%:R"
                        icacls "%SSH_KEY_FILE%"

                        echo.
                        echo Running SSH test...

                        ssh -i "%SSH_KEY_FILE%" ^
                            -o StrictHostKeyChecking=no ^
                            -o UserKnownHostsFile=NUL ^
                            -o IdentitiesOnly=yes ^
                            -o BatchMode=yes ^
                            %SSH_USER%@${params.TARGET_HOST} "hostname && whoami && sudo -n whoami"
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
            echo 'SSH connection test failed. Check Jenkins credentials, VM IP, SSH service, sudo permissions, or Windows key file permissions.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}