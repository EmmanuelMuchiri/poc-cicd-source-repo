pipeline {
    agent any

    environment {
        // Paths and credentials
        APICTL_PATH = '/usr/local/bin/apictl'
        APIM_ENV = 'dev'
        APIM_USER = 'admin'
        APIM_PASS = 'admin'

        // Ensure /usr/local/bin is in PATH
        PATH = "/usr/local/bin:${env.PATH}"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "===== Printing PATH ====="
                echo $PATH
                echo "===== Checking apictl in PATH ====="
                which apictl || echo "apictl not found in PATH"
                echo "===== Checking apictl version ====="
                apictl version || echo "apictl not executable"
                '''
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/EmmanuelMuchiri/poc-cicd-source-repo.git'
            }
        }

        stage('Login to API Manager') {
            steps {
                sh """
                echo "===== Logging into API Manager ====="
                ${APICTL_PATH} login ${APIM_ENV} -u ${APIM_USER} -p ${APIM_PASS} --insecure
                """
            }
        }

        stage('Deploy API') {
            steps {
                sh """
                echo "===== Deploying API ====="
                ${APICTL_PATH} import api \\
                    -f PetstoreAPI \\
                    --environment ${APIM_ENV} \\
                    --update \\
                    --preserve-provider \\
                    --insecure
                """
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
