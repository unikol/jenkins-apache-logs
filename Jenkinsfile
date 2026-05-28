pipeline {
    agent any

    parameters {
        string(
            name: 'TARGET_HOST',
            defaultValue: '192.168.56.102',
            description: 'Target Linux VM IP address'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Repository was checked out successfully.'
            }
        }

        stage('Prepare deployment scripts') {
            steps {
                writeFile file: 'deploy_apache.sh', text: '''#!/usr/bin/env bash
set -euo pipefail

echo "=== Operating system information ==="
cat /etc/os-release

echo "=== Detecting package manager ==="

if command -v apt-get >/dev/null 2>&1; then
    WEB_PACKAGE="apache2"
    WEB_SERVICE="apache2"

    echo "Detected Debian/Ubuntu based system"
    sudo apt-get update -y
    sudo DEBIAN_FRONTEND=noninteractive apt-get install -y apache2

elif command -v dnf >/dev/null 2>&1; then
    WEB_PACKAGE="httpd"
    WEB_SERVICE="httpd"

    echo "Detected dnf based system"
    sudo dnf install -y httpd

elif command -v yum >/dev/null 2>&1; then
    WEB_PACKAGE="httpd"
    WEB_SERVICE="httpd"

    echo "Detected yum based system"
    sudo yum install -y httpd

else
    echo "Unsupported Linux distribution. No apt-get, dnf or yum found."
    exit 1
fi

echo "=== Enabling and starting web server ==="
sudo systemctl enable "$WEB_SERVICE"
sudo systemctl restart "$WEB_SERVICE"

echo "=== Creating test web page ==="
echo "<h1>Deployed by Jenkins Pipeline</h1><p>Apache/httpd installation completed successfully.</p>" | sudo tee /var/www/html/index.html >/dev/null

echo "=== Checking service status ==="
sudo systemctl status "$WEB_SERVICE" --no-pager

echo "=== Checking HTTP response ==="
curl -I http://localhost

echo "=== Deployment completed successfully ==="
'''

                writeFile file: 'check_logs.sh', text: '''#!/usr/bin/env bash
set -euo pipefail

echo "=== Searching for Apache/httpd access log ==="

if [ -f /var/log/apache2/access.log ]; then
    LOG_FILE="/var/log/apache2/access.log"
elif [ -f /var/log/httpd/access_log ]; then
    LOG_FILE="/var/log/httpd/access_log"
else
    echo "No Apache/httpd access log file found."
    exit 2
fi

echo "Using log file: $LOG_FILE"

echo "=== Last 20 access log lines ==="
sudo tail -20 "$LOG_FILE"

echo "=== Checking for 4xx and 5xx HTTP status codes ==="

ERROR_LINES=$(sudo awk '$9 ~ /^[45][0-9][0-9]$/ {print}' "$LOG_FILE" || true)

if [ -n "$ERROR_LINES" ]; then
    echo "Found 4xx or 5xx errors:"
    echo "$ERROR_LINES"
    exit 1
else
    echo "No 4xx or 5xx errors found in the access log."
    exit 0
fi
'''
            }
        }

        stage('Test SSH connection') {
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

                        echo Target host: ${params.TARGET_HOST}
                        echo SSH user: %SSH_USER%

                        for /f "delims=" %%U in ('whoami') do set "CURRENT_USER=%%U"

                        icacls "%SSH_KEY_FILE%" /inheritance:r
                        icacls "%SSH_KEY_FILE%" /grant:r "%CURRENT_USER%:R"

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

        stage('Install and start Apache/httpd') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'jenkins-apache-ssh-key',
                        keyFileVariable: 'SSH_KEY_FILE',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    bat """
                        echo Fixing SSH private key permissions...

                        for /f "delims=" %%U in ('whoami') do set "CURRENT_USER=%%U"

                        icacls "%SSH_KEY_FILE%" /inheritance:r
                        icacls "%SSH_KEY_FILE%" /grant:r "%CURRENT_USER%:R"

                        echo Copying deployment script to remote VM...

                        scp -i "%SSH_KEY_FILE%" ^
                            -o StrictHostKeyChecking=no ^
                            -o UserKnownHostsFile=NUL ^
                            -o IdentitiesOnly=yes ^
                            -o BatchMode=yes ^
                            deploy_apache.sh %SSH_USER%@${params.TARGET_HOST}:/tmp/jenkins_deploy_apache.sh

                        echo Running deployment script on remote VM...

                        ssh -i "%SSH_KEY_FILE%" ^
                            -o StrictHostKeyChecking=no ^
                            -o UserKnownHostsFile=NUL ^
                            -o IdentitiesOnly=yes ^
                            -o BatchMode=yes ^
                            %SSH_USER%@${params.TARGET_HOST} "tr -d '\\r' < /tmp/jenkins_deploy_apache.sh > /tmp/jenkins_deploy_apache_fixed.sh && sudo bash /tmp/jenkins_deploy_apache_fixed.sh"
                    """
                }
            }
        }

        stage('Check web server logs for 4xx and 5xx') {
            steps {
                script {
                    def logCheckStatus = 0

                    withCredentials([
                        sshUserPrivateKey(
                            credentialsId: 'jenkins-apache-ssh-key',
                            keyFileVariable: 'SSH_KEY_FILE',
                            usernameVariable: 'SSH_USER'
                        )
                    ]) {
                        logCheckStatus = bat(
                            script: """
                                echo Fixing SSH private key permissions...

                                for /f "delims=" %%U in ('whoami') do set "CURRENT_USER=%%U"

                                icacls "%SSH_KEY_FILE%" /inheritance:r
                                icacls "%SSH_KEY_FILE%" /grant:r "%CURRENT_USER%:R"

                                echo Copying log check script to remote VM...

                                scp -i "%SSH_KEY_FILE%" ^
                                    -o StrictHostKeyChecking=no ^
                                    -o UserKnownHostsFile=NUL ^
                                    -o IdentitiesOnly=yes ^
                                    -o BatchMode=yes ^
                                    check_logs.sh %SSH_USER%@${params.TARGET_HOST}:/tmp/jenkins_check_logs.sh

                                echo Running log check script on remote VM...

                                ssh -i "%SSH_KEY_FILE%" ^
                                    -o StrictHostKeyChecking=no ^
                                    -o UserKnownHostsFile=NUL ^
                                    -o IdentitiesOnly=yes ^
                                    -o BatchMode=yes ^
                                    %SSH_USER%@${params.TARGET_HOST} "tr -d '\\r' < /tmp/jenkins_check_logs.sh > /tmp/jenkins_check_logs_fixed.sh && sudo bash /tmp/jenkins_check_logs_fixed.sh"
                            """,
                            returnStatus: true
                        )
                    }

                    if (logCheckStatus == 1) {
                        currentBuild.result = 'UNSTABLE'
                        echo '4xx or 5xx errors were found in the Apache/httpd access log. Build marked as UNSTABLE.'
                    } else if (logCheckStatus == 2) {
                        error 'Apache/httpd access log file was not found.'
                    } else if (logCheckStatus != 0) {
                        error "Unexpected error during log verification. Exit code: ${logCheckStatus}"
                    } else {
                        echo 'Log verification completed successfully. No 4xx/5xx errors found.'
                    }
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully. Apache/httpd is installed and logs were checked.'
        }

        unstable {
            echo 'Deployment completed, but 4xx/5xx errors were found in the logs.'
        }

        failure {
            echo 'Pipeline failed. Check Jenkins console output for details.'
        }

        always {
            echo 'Pipeline execution finished.'
        }
    }
}