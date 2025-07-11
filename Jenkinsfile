pipeline {
    agent any
 
    environment {
        APICTL_PATH = '/usr/local/bin/apictl'
        APIM_ENV = 'dev'
        APIM_USER = 'admin'
        APIM_PASS = 'admin'
    }
 
    stages {
 
        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/EmmanuelMuchiri/poc-cicd-source-repo.git'
            }
        }
 
        stage('Login to API Manager') {
            steps {
                sh '''
                ${APICTL_PATH} login ${APIM_ENV} -u ${APIM_USER} -p ${APIM_PASS} --insecure
                '''
            }
        }
 
        stage('Deploy API') {
            steps {
                sh '''
                ${APICTL_PATH} import-api -f your-exported-api-folder --environment ${APIM_ENV} --update --preserve-provider --skip-cleanup --insecure
                '''
            }
        }
    }
}

docker cp "C:\Users\Lenovo\Downloads\apictl-4.4.1-linux-amd64.tar.gz" bold_galileo:/tmp/apictl-4.4.1-linux-amd64.tar.gz
