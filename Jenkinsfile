#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@AWS-Practice', retriever: modernSCM(
    [
        $class: 'GitSCMSource',
        remote: 'https://gitlab.com/saad324/jenkins-shared-library.git',
        credentialsId: 'saad-git'
    ]
)

pipeline {
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        AWS_ECR_SERVER= '649513566336.dkr.ecr.eu-west-2.amazonaws.com'
        AWS_ECR= "${AWS_ECR_SERVER}/my-app" 
    }
   

    stages {
        stage('increment version') {
            steps {
                script {
                    echo 'incrementing app version...'
                    sh 'mvn build-helper:parse-version versions:set \
                        -DnewVersion=\\\${parsedVersion.majorVersion}.\\\${parsedVersion.minorVersion}.\\\${parsedVersion.nextIncrementalVersion} \
                        versions:commit'
                    def matcher = readFile('pom.xml') =~ '<version>(.+)</version>'
                    def version = matcher[0][1]
                    env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }

        stage("build and push image") {
            steps {
                 script {
                   echo "building the docker image..."
withCredentials([usernamePassword(credentialsId: 'Ecr-for-like-cluster', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                sh "docker build -t ${AWS_ECR}:${IMAGE_NAME} ."
                sh "echo $PASSWORD | docker login -u $USERNAME --password-stdin ${AWS_ECR_SERVER}"
                sh "docker push ${AWS_ECR}:${IMAGE_NAME}"
                    }
                }
            }
        }




        stage("test") {
            steps {
                script {
                    echo "testing the app"
                }
            }
        }

        stage('deploy') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'lke-cluster-key', serverUrl: 'https://d47bed48-5b8f-4294-bcf0-afb6d7e373f5.eu-west-2.linodelke.net']){
            
                        sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'
                        sh 'envsubst < kubernetes/deployment.yaml | kubectl apply -f -'

                    }
                }
            }
        }

        stage('commit update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'saad-git', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'git config --global user.email "saadi57214@gmail.com"'
                        sh 'git config --global user.name "saad324"'

                        sh 'git status'
                        sh 'git branch'
                        sh 'git config --list'

                        sh "git remote set-url origin https://${USERNAME}:${PASSWORD}@gitlab.com/saad324/k8s-projects.git"

                        sh 'git add .'
                        sh 'git commit -m "version update"'
                        sh 'git push origin HEAD:k8s-CI/CD-pipeline'
                    }
                }
            }
        }
    }
}
