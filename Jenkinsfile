#! /usr/bin/env groovy
@Library('jenkins-shared-library')
def gv

pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
    environment {
        IMAGE_NAME = 'irschad/java-app:3.0'
    }
    
    stages {
        stage('build app') {
            steps {
                echo 'building application jar...'
                buildJar()
            }
        }
        stage('build image') {
            steps {
                script {
                    echo 'building the docker image...'
                    buildImage(env.IMAGE_NAME)
                    dockerLogin()
                    dockerPush(env.IMAGE_NAME)
                }
            }
        } 
        stage("deploy") {
            steps {
                script {
                    echo 'deploying docker image to EC2...'
                    def dockerCmd = "docker run -p 8080:8080 -d ${IMAGE_NAME}"
                    sshagent(['aws-ec2-server-key']) {
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@34.229.224.203 ${dockerCmd}"
                    }
                }
            }               
        }
    }
} 
