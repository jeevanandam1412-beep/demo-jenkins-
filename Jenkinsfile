pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/usr/share/nginx/html'
        SOURCE_DIR = 'dist'
        BACKUP_DIR = '/var/backups/nginx-deploy'
        EMAIL_TO = 'jeevanandam1412@gmail.com'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from GitHub...'

                checkout scm
            }
        }

        stage('Backup Current Website') {
            steps {
                sh '''
                    set -e

                    echo "Creating backup directory..."

                    sudo mkdir -p "$BACKUP_DIR"

                    echo "Creating deployment backup..."

                    sudo rm -rf "$BACKUP_DIR/latest"
                    sudo mkdir -p "$BACKUP_DIR/latest"

                    if [ -d "$DEPLOY_DIR" ]; then
                        sudo cp -a "$DEPLOY_DIR/." "$BACKUP_DIR/latest/"
                    fi

                    echo "Backup completed successfully."
                '''
            }
        }

        stage('Deploy Website Files') {
            steps {
                sh '''
                    set -e

                    echo "Checking source directory..."

                    if [ ! -d "$SOURCE_DIR" ]; then
                        echo "ERROR: $SOURCE_DIR directory does not exist"
                        exit 1
                    fi

                    echo "Deploying website files..."

                    sudo rm -rf "$DEPLOY_DIR"/*

                    sudo cp -a "$SOURCE_DIR/." "$DEPLOY_DIR/"

                    echo "Website files deployed successfully."
                '''
            }
        }

        stage('Nginx Configuration Test') {
            steps {
                sh '''
                    set -e

                    echo "Testing Nginx configuration..."

                    sudo nginx -t

                    echo "Nginx configuration test passed."
                '''
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input(
                        message: 'Nginx configuration test passed. Do you want to continue with the deployment?',
                        ok: 'Approve Deployment'
                    )
                }
            }
        }

        stage('Restart Nginx') {
            steps {
                sh '''
                    set -e

                    echo "Restarting Nginx..."

                    sudo systemctl restart nginx

                    echo "Checking Nginx status..."

                    sudo systemctl is-active --quiet nginx

                    echo "Nginx restarted successfully."
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "Performing health check..."

                    curl -f http://localhost/

                    echo ""
                    echo "Health check successful."
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'Nginx deployment completed successfully'
            echo '======================================'

            emailext(
                subject: "SUCCESS: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment Successful

Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Server    : ${env.NODE_NAME}
Status    : SUCCESS

GitHub code was pulled successfully.

Files deployed to:
${DEPLOY_DIR}

Nginx configuration test passed.
Manual approval was received.
Nginx was restarted successfully.
Health check passed.

Build URL:
${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }

        failure {
            echo '======================================'
            echo 'Nginx deployment FAILED'
            echo '======================================'

            emailext(
                subject: "FAILED: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment FAILED

Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Server    : ${env.NODE_NAME}
Status    : FAILED

The deployment did not complete successfully.

Possible reasons:

- GitHub checkout failed
- Source directory missing
- File copy failed
- Nginx configuration test failed
- Nginx restart failed
- Health check failed

Build URL:
${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }

        aborted {
            echo '======================================'
            echo 'Nginx deployment was ABORTED'
            echo '======================================'

            emailext(
                subject: "ABORTED: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment ABORTED

Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Server    : ${env.NODE_NAME}
Status    : ABORTED

The deployment was cancelled or the manual approval timed out.

Build URL:
${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }
    }
}