```groovy
pipeline {
    agent any

    environment {
        DEPLOY_DIR = '/var/www/html'
        SOURCE_DIR = 'dist'
        BACKUP_DIR = '/var/backups/nginx-deploy'
        EMAIL_TO = 'your-email@example.com'
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from GitHub...'

                checkout scm
            }
        }

        stage('Prepare Deployment') {
            steps {
                sh '''
                    set -e

                    echo "Checking source directory..."

                    if [ ! -d "$SOURCE_DIR" ]; then
                        echo "ERROR: $SOURCE_DIR directory does not exist"
                        exit 1
                    fi

                    echo "Source directory found:"
                    ls -la "$SOURCE_DIR"

                    echo "Creating backup directory..."

                    sudo mkdir -p "$BACKUP_DIR"

                    echo "Creating deployment backup..."

                    sudo rm -rf "$BACKUP_DIR/latest"

                    sudo mkdir -p "$BACKUP_DIR/latest"

                    if [ -d "$DEPLOY_DIR" ]; then
                        sudo cp -a "$DEPLOY_DIR/." "$BACKUP_DIR/latest/"
                    fi

                    echo "Backup completed."
                '''
            }
        }

        stage('Nginx Configuration Test') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "Testing Nginx configuration..."
                    echo "========================================"

                    sudo nginx -t

                    echo "Nginx configuration test PASSED."
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

        stage('Deploy Files') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "Deploying files to Nginx..."
                    echo "========================================"

                    sudo rm -rf "$DEPLOY_DIR"/*

                    sudo cp -a "$SOURCE_DIR/." "$DEPLOY_DIR/"

                    echo "Files copied successfully."

                    echo "Deployed files:"
                    ls -la "$DEPLOY_DIR"
                '''
            }
        }

        stage('Restart Nginx') {
            steps {
                sh '''
                    set -e

                    echo "========================================"
                    echo "Restarting Nginx..."
                    echo "========================================"

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

                    echo "========================================"
                    echo "Performing health check..."
                    echo "========================================"

                    curl -f http://localhost/

                    echo ""
                    echo "Health check successful."
                '''
            }
        }
    }

    post {

        success {
            echo '========================================'
            echo 'Nginx deployment completed successfully.'
            echo '========================================'

            emailext(
                subject: "SUCCESS: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment Successful

Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Branch    : ${env.BRANCH_NAME}
Server    : ${env.NODE_NAME}
Status    : SUCCESS

GitHub code was pulled successfully.

Files were deployed to:
${DEPLOY_DIR}

Source directory:
${SOURCE_DIR}

nginx -t:
PASSED

Manual approval:
APPROVED

Nginx restart:
SUCCESSFUL

Health check:
PASSED

Build URL:
${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }

        failure {
            echo '========================================'
            echo 'Nginx deployment failed.'
            echo '========================================'

            emailext(
                subject: "FAILED: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment FAILED

Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Branch    : ${env.BRANCH_NAME}
Server    : ${env.NODE_NAME}
Status    : FAILED

The deployment did not complete successfully.

Possible reasons:

- GitHub checkout failed
- Source directory does not exist
- Backup failed
- nginx -t failed
- File copy failed
- Nginx restart failed
- Health check failed

Please check the Jenkins console output.

Build URL:
${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }

        aborted {
            echo '========================================'
            echo 'Nginx deployment was aborted.'
            echo '========================================'

            emailext(
                subject: "ABORTED: Nginx Deployment - ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
Nginx Deployment ABORTED

Job       : ${env.JOB_NAME}
Build     : #${env.BUILD_NUMBER}
Branch    : ${env.BRANCH_NAME}
Server    : ${env.NODE_NAME}
Status    : ABORTED

The deployment was cancelled or the manual approval was rejected/timed out.

No deployment was performed after the approval stage.

Build URL:
${env.BUILD_URL}
""",
                to: "${EMAIL_TO}"
            )
        }
    }
}
```
