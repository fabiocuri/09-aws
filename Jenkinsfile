#!/usr/bin/env groovy

library identifier: 'jenkins-shared-library@master', retriever: modernSCM(
    [$class: 'GitSCMSource',
    remote: 'https://gitlab.com/fabiocuri/jenkins-shared-library.git',
    credentialsID: 'gitlab_credentials'
    ]
)

pipeline {
    agent any
      tools {
        nodejs "node"
      }
      stages {
        stage('increment version') {
            node {
                sh 'npm version patch'
                def version = readFile('./app/package.json').toString().parseJson().version
                env.IMAGE_NAME = version + '-' + BUILD_NUMBER
                env.REPO_NAME = 'fabiocuri/aws-npm-exercise:' + env.IMAGE_NAME
            }
        }
        stage('Build and Push docker image') {
            steps {
                script {
                    buildNpm()
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME, env.REPO_NAME)
                }
            }


        }
        stage('deploy to EC2') {
            steps {
                script {
                   def shellCmd = "bash ./server-cmds.sh ${IMAGE_NAME}"
                   def ec2Instance = "ec2-user@public-ip-address"

                   sshagent(['ec2-server-key']) {
                       sh "scp -o StrictHostKeyChecking=no server-cmds.sh ${ec2Instance}:/home/ec2-user"
                       sh "scp -o StrictHostKeyChecking=no docker-compose.yaml ${ec2Instance}:/home/ec2-user"
                       sh "ssh -o StrictHostKeyChecking=no ${ec2Instance} ${shellCmd}"
                   }     
                }
            }
        }
        stage('commit version update') {
          ...
        }
    }     
}
