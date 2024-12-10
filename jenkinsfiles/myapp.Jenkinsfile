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
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()    // Приписывает timestamp к шагам
    }
    stages {
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
        // stage('Cloning Git') {
        //     steps {
        //         git 'https://github.com/ValentinStupa/diplom_images.git'
        //     }
        // }
        stage('Building image') {
            steps{
                script {
                     dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy Image') {
        steps{
            script {
            docker.withRegistry( '', registryCredential ) {
                dockerImage.push()
            }
            }
        }
        }
        stage('Remove Unused docker image') {
        steps{
            sh "docker rmi $registry:$BUILD_NUMBER"
        }
        }
    }
}
