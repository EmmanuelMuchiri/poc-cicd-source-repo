pipeline {
    agent any

    environment {
        APICTL_PATH = '/usr/local/bin/apictl'
        APIM_ENV = 'dev'
        APIM_USER = 'admin'
        APIM_PASS = 'admin'
        PATH = "/usr/local/bin:${env.PATH}" // Ensures /usr/local/bin is in PATH
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "Printing PATH:"
                echo $PATH
                echo "Checking apictl in PATH:"
                which apictl || echo "apictl not found in PATH"
                echo "Listing /usr/local/bin:"
                ls -l /usr/local/bin
                echo "Checking apictl version:"
                /usr/local/bin/apictl version || echo "apictl not executable"
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
                ${APICTL_PATH} login ${APIM_ENV} -u ${APIM_USER} -p ${APIM_PASS} --insecure
                """
            }
        }

        stage('Deploy API') {
            steps {
                sh """
                ${APICTL_PATH} import-api -f your-exported-api-folder --environment ${APIM_ENV} --update --preserve-provider --skip-cleanup --insecure
                """
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed. Check logs for details."
        }
        success {
            echo "Pipeline executed successfully."
        }
    }
}
