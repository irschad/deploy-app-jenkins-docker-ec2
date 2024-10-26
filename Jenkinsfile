#! /usr/bin/env groovy
@Library('jenkins-shared-library')
def gv

pipeline {   
    agent any
    tools {
        maven 'Maven'
    }
    stages {
        stage("init") {
            steps {
                script {
                    gv = load "script.groovy"
                    echo "Executing pipeline for branch $BRANCH_NAME"
                }
            }
        }
        stage("build jar") {
            steps {
                script {
                     buildJar()

                }
            }
        }

        stage("build and push image") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }
            steps {
                script {
                    buildImage 'irschad/java-app:3.0'
                    dockerLogin()
                    dockerPush 'irschad/java-app:3.0'
                }
            }
        }

        stage("deploy") {
            when {
                expression {
                    BRANCH_NAME == "master"
                }
            }
            steps {
                script {
                    gv.deployApp()
                }
            }
        }               
    }
} 
