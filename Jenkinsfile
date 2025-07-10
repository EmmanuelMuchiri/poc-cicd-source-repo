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
        APICTL_PATH = '/usr/local/bin'
        PATH = "${APICL_PATH}:${env.PATH}"
    }

    stages {
        
        stage('Preparation') {
            steps{
                git branch: "master",
                url: 'https://github.com/EmmanuelMuchiri/poc-cicd-source-repo.git'
            }
        }

        stage('Setup Environment for APICTL') {
            steps {
                sh '''#!/bin/bash
                export PATH=/usr/local/bin:$PATH

                apictl set --vcs-config-path /var/lib/jenkins/workspace/gitconfig

                envs=$(apictl get envs --format "{{.Name}}")
                if [ -z "$envs" ]; 
                then 
                    echo "No environment configured. Setting dev environment.."
                    apictl add env dev --apim https://${APIM_DEV_HOST}:9443 
                else
                    echo "Environments :"$envs
                    if [[ $envs != *"dev"* ]]; then
                        echo "Dev environment is not configured. Setting dev environment.."
                        apictl add env dev --apim https://${APIM_DEV_HOST}:9443 
                    fi
                fi
                '''
            }
        }

        stage('Build api bundles and upload') {
            steps {
                sh '''#!/bin/bash
                export PATH=/usr/local/bin:$PATH

                apictl login dev -u admin -p admin -k

                apis=$(apictl vcs status -e dev --format="{{ jsonPretty . }}" | jq -r '.API | .[] | .NickName')
                mkdir -p upload
                if [ -z "$apis" ]; 
                then 
                    echo "======== No API Changes detected =========="; 
                else 
                    echo "Updated APIs :"$apis
                    apiArray=($apis)
                    for i in "${apiArray[@]}"
                    do
                        echo "$i"
                        apictl bundle -s $i -d upload

                        versionFull=$(cat $i/meta.yaml)
                        versionId=(${versionFull//: / })
                        version=${versionId[1]}

                        for file in upload/*; do
                            echo "Uploading "$file
                            curl -u ${ARTIFACTORY_USER}:${ARTIFACTORY_PWD} -X PUT https://${ARTIFACTORY_HOST}/artifactory/${ARTIFACTORY_REPO}/$i/$version/ -T $file
                        done
                        rm -rf upload
                    done
                fi

                '''
            }
        }

        stage('Update local repo') {
            steps {
                sh '''#!/bin/bash
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
