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
        REMOTE_HTML = '/var/www/html/index.html'
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

                        REM Backup existing remote index.html (if any) to .previous
                        ssh -o StrictHostKeyChecking=no -i "%KEYFILE%" %REMOTE_USER%@%REMOTE_HOST% ^
                          "if [ -f ${REMOTE_HTML} ]; then sudo cp ${REMOTE_HTML} ${REMOTE_HTML}.previous; fi"

                        REM Deploy new index.html
                        scp -o StrictHostKeyChecking=no -i "%KEYFILE%" index.html %REMOTE_USER%@%REMOTE_HOST%:%REMOTE_DIR%/

                        REM Move file into /var/www/html (served by nginx)
                        ssh -o StrictHostKeyChecking=no -i "%KEYFILE%" %REMOTE_USER%@%REMOTE_HOST% ^
                          "sudo cp %REMOTE_DIR%/index.html ${REMOTE_HTML} && sudo chown www-data:www-data ${REMOTE_HTML} && sudo chmod 644 ${REMOTE_HTML}"
                    """
                }
            }
         }

    }

    post {
        failure {
            echo "Deployment failed. Rolling back to previous index.html..."
            withCredentials([sshUserPrivateKey(credentialsId: "${env.SSH_KEY_ID}", keyFileVariable: 'KEYFILE')]) {
                bat """
                    echo Restoring previous version using SSH key: %KEYFILE%

                    ssh -o StrictHostKeyChecking=no -i "%KEYFILE%" %REMOTE_USER%@%REMOTE_HOST% ^
                      "if [ -f ${REMOTE_HTML}.previous ]; then sudo cp ${REMOTE_HTML}.previous ${REMOTE_HTML} && sudo chown www-data:www-data ${REMOTE_HTML} && sudo chmod 644 ${REMOTE_HTML}; else echo 'No previous version to roll back to'; fi"
                """
            }
        }
    }
}
