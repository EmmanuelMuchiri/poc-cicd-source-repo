pipeline {

    agent {
        node {
            label 'master' 
        }
    }

    options {
        buildDiscarder logRotator( 
            daysToKeepStr: '16', 
            numToKeepStr: '10'
        )
    }

    environment {
        APICTL_PATH = "${env.WORKSPACE}/apictl"
        PATH = "${APICTL_PATH}:${env.PATH}"
        APIM_HOST = "localhost" // change if your API Manager runs on a different host
        APIM_PORT = "9443"
        APIM_USER = "admin"
        APIM_PASS = "admin"
    }

    stages {

        stage('Preparation') {
            steps {
                git branch: "master",
                url: 'https://github.com/EmmanuelMuchiri/poc-cicd-source-repo.git'
            }
        }

        stage('Install APICTL CLI') {
            steps {
                sh '''
                wget https://github.com/wso2/product-apim-tooling/releases/download/v4.2.6/apictl-linux-x64 -O apictl
                chmod +x apictl
                ./apictl version
                '''
            }
        }

        stage('Setup Environment for APICTL') {
            steps {
                sh '''
                ./apictl set --vcs-config-path /var/lib/jenkins/workspace/gitconfig

                envs=$(./apictl get envs --format "{{.Name}}")
                if [ -z "$envs" ]; 
                then 
                    echo "No environment configured. Setting dev environment.."
                    ./apictl add env dev --apim https://${APIM_HOST}:${APIM_PORT}
                else
                    echo "Environments :"$envs
                    if [[ $envs != *"dev"* ]]; then
                        echo "Dev environment is not configured. Setting dev environment.."
                        ./apictl add env dev --apim https://${APIM_HOST}:${APIM_PORT}
                    fi
                fi
                '''
            }
        }

        stage('Build api bundles and deploy to local APIM') {
            steps {
                sh '''
                ./apictl login dev -u ${APIM_USER} -p ${APIM_PASS} -k

                apis=$(./apictl vcs status -e dev --format="{{ jsonPretty . }}" | jq -r '.API | .[] | .NickName')
                mkdir -p upload
                if [ -z "$apis" ]; 
                then 
                    echo "======== No API Changes detected =========="; 
                else 
                    echo "Updated APIs :"$apis
                    apiArray=($apis)
                    for i in "${apiArray[@]}"
                    do
                        echo "Building bundle for $i"
                        ./apictl bundle -s $i -d upload

                        echo "Deploying $i to local API Manager"
                        ./apictl import api -f upload/$i -e dev -k
                    done
                fi
                '''
            }
        }

        stage('Update local repo') {
            steps {
                sh '''
                idFull=$(cat vcs.yaml)
                arrId=(${idFull//: / })
                repoId=${arrId[1]}
                head=$(git rev-parse HEAD)

                rm /var/lib/jenkins/workspace/gitconfig
                echo "
repos:
  $repoId:
    environments:
       dev:
        lastAttemptedRev: $head" >> /var/lib/jenkins/workspace/gitconfig
                '''
            }
        }
    }
}
