pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/var/www/html'
        SOURCE_DIR = 'dist'
        BACKUP_DIR = '/var/backups/nginx-deploy'
        EMAIL_TO   = 'your-email@gmail.com'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Pulling code from GitHub...'

                checkout scm

                sh '''
                    set -e

                    echo "Current commit:"
                    git rev-parse HEAD

                    echo "Checking source directory..."

                    if [ ! -d "$SOURCE_DIR" ]; then
                        echo "ERROR: dist directory does not exist!"
                        exit 1
                    fi

                    echo "Source files:"
                    ls -la "$SOURCE_DIR"
                '''
            }
        }

        stage('Backup Current Website') {
            steps {
                sh '''
                    set -e

                    echo "Creating backup directory..."

                    sudo mkdir -p "$BACKUP_DIR"

                    echo "Creating backup..."

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

                    echo "Deploying website files..."

                    sudo mkdir -p "$DEPLOY_DIR"

                    sudo rm -rf "$DEPLOY_DIR"/*

                    sudo cp -a "$SOURCE_DIR/." "$DEPLOY_DIR/"

                    echo "Deployment files copied successfully."

                    echo "Files currently in Nginx directory:"
                    sudo ls -la "$DEPLOY_DIR"
                '''
            }
        }

        stage('Nginx Configuration Test') {
            steps {
                sh '''
                    set -e

                    echo "Testing Nginx configuration..."

                    sudo nginx -t

                    echo "Nginx configuration test PASSED."
                '''
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input(
                        message: 'nginx -t passed. Approve deployment and restart Nginx?',
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

                    echo "Nginx is running successfully."
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    set -e

                    echo "Running website health check..."

                    curl -f http://localhost/

                    echo ""
                    echo "Website health check PASSED."
                '''
            }
        }
    }

    post {

        success {
            echo '======================================'
            echo 'Nginx deployment SUCCESSFUL'
            echo '======================================'

            emailext(
                to: "${EMAIL_TO}",
                subject: "SUCCESS: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment Successful

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Server: ${env.NODE_NAME}
Status: SUCCESS

GitHub code was pulled successfully.

Source:
${SOURCE_DIR}

Deployment directory:
${DEPLOY_DIR}

Nginx configuration test:
PASSED

Manual approval:
APPROVED

Nginx restart:
SUCCESSFUL

Health check:
PASSED

Build URL:
${env.BUILD_URL}
"""
            )
        }

        failure {
            echo '======================================'
            echo 'Nginx deployment FAILED'
            echo '======================================'

            emailext(
                to: "${EMAIL_TO}",
                subject: "FAILED: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment FAILED

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Server: ${env.NODE_NAME}
Status: FAILED

Please check the Jenkins Console Output.

Build URL:
${env.BUILD_URL}
"""
            )
        }

        aborted {
            echo '======================================'
            echo 'Nginx deployment ABORTED'
            echo '======================================'

            emailext(
                to: "${EMAIL_TO}",
                subject: "ABORTED: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment ABORTED

Job: ${env.JOB_NAME}
Build: #${env.BUILD_NUMBER}
Server: ${env.NODE_NAME}
Status: ABORTED

The deployment was cancelled or the approval timed out.

Build URL:
${env.BUILD_URL}
"""
            )
        }
    }
}