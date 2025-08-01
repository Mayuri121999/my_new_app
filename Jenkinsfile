pipeline {
    agent any

    parameters {
        string(name: 'DEPLOY_VERSION', defaultValue: 'v1.0.0', description: 'Version for this deployment')
        string(name: 'BRANCH_NAME', defaultValue: 'dev', description: 'Git branch to deploy')
        choice(name: 'ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
    }

    environment {
        SSH_KEY_ID = 'ec2-ssh-key' // Jenkins SSH credential ID
        REMOTE_USER = 'ubuntu'
        REMOTE_HOST = '51.20.181.213'
        REMOTE_DIR = 'my_new_app'
        // PORT = '3000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${params.BRANCH_NAME}"]],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Mayuri121999/my_new_app.git'
                    ]]
                ])
            }
        }

        stage('Deploy') {
            steps {
                echo "Deploying branch ${params.BRANCH_NAME} to ${params.ENV}"

                withCredentials([sshUserPrivateKey(credentialsId: "${env.SSH_KEY_ID}", keyFileVariable: 'KEYFILE')]) {
                    bat """
                        echo Using SSH key: %KEYFILE%

                        REM Secure key
                        icacls "%KEYFILE%" /inheritance:r
                        icacls "%KEYFILE%" /grant:r "SYSTEM:F"

                        REM Check key file
                        dir "%KEYFILE%"

                        REM Deploy file to EC2
                        scp -o StrictHostKeyChecking=no -i "%KEYFILE%" index.html %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/

                        REM Move file into /var/www/html (served by nginx)
                        ssh -o StrictHostKeyChecking=no -i "%KEYFILE%" %REMOTE_USER%@%REMOTE_HOST% ^
                          "sudo cp %REMOTE_DIR%/index.html /var/www/html/index.html && sudo chown www-data:www-data /var/www/html/index.html && sudo chmod 644 /var/www/html/index.html"
                    """
                }
            }
         }

    }

    post {
        failure {
            echo "Deployment failed. Rolling back..."
        }
    }
}