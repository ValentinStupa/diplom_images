#!groovy

/* groovylint-disable-next-line CompileStatic */
properties([disableConcurrentBuilds()])     // Не позволяет одновременно запускать несколько сборок

pipeline {
    environment {
        registry = 'valentinstupa/custom-nginx'
        registryCredential = 'dockerhub_access'
        dockerImage = ''
    }
    // agent {
    //     label 'main' // Собирать только по этой ветке
    // }
    agent any
    options {
        /* groovylint-disable-next-line DuplicateStringLiteral */
        buildDiscarder(logRotator(numToKeepStr: '5', artifactNumToKeepStr: '5'))
        timestamps()    // Приписывает timestamp к шагам
    }
    stages {
        // stage("Get last git tag") {
        //     steps {
        //         script {
        //             def git_tag = sh(returnStdout: true, script: "git tag --list | tail -1").trim()
        //             echo "Git tag: $git_tag"
        //             env.tag = git_tag
        //         } 
        //     }
        // }
        stage("clean workspace") {
            steps {
                script {
                    deleteDir()
                }
             }
        }
        stage('Checkout docker repo') {
            steps {
                checkout scmGit(branches: [[name: 'main']],
                userRemoteConfigs: [[url: 'https://github.com/ValentinStupa/diplom_images.git']])
            }
        }
        stage('Add tag to index.html') {
            steps {
                script {
                    sh """
                        sed -i '\$d' index.html
                        sed -i -e '\$a<tagname>$BUILD_NUMBER</tagname>' index.html
                        """                        
                }   
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy Image') {
            steps {
                script {
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Remove Unused docker image') {
            steps {
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

    }
}
