#!groovy

/* groovylint-disable-next-line CompileStatic */
properties([disableConcurrentBuilds()])     // Не позволяет одновременно запускать несколько сборок

pipeline {
    environment {
        registry = 'valentinstupa/custom-nginx'
        registryCredential = 'dockerhub_access'
        dockerImage = ''
        KUBECONFIG = 'kube_config'
        manifest = "manifest_files/myapp/nginx_deploy.yml"
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
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":1.0.$BUILD_NUMBER"
                }
            }
        }
        // stage('Deploy Image') {
        //     steps {
        //         script {
        //             docker.withRegistry('', registryCredential) {
        //                 dockerImage.push()
        //             }
        //         }
        //     }
        // }
        stage('Remove Unused docker image') {
            steps {
                sh "docker rmi $registry:1.0.$BUILD_NUMBER"
            }
        }
//      
//  Work with K8s cluster
//
        stage('Checkout K8s repo') {
            steps {
                checkout scmGit(branches: [[name: 'main']],
                userRemoteConfigs: [[url: 'https://github.com/ValentinStupa/diplom_k8s.git']])
            }
        }
        stage('Get PREVIOUS_BUILD_NUMBER') {
            steps {
                script {
                    PREVIOUS_BUILD_NUMBER = sh(returnStdout: true, script: 'grep "image:" ${manifest}|awk -F \":\" \'{print $3}\'')
                    echo "Output: ${PREVIOUS_BUILD_NUMBER}"
                    env.pre_build = PREVIOUS_BUILD_NUMBER
                }                           
            }
        }
        stage('Change build number') {
                steps {
                    script {
                        // Set KUBECONFIG environment variable
                        withEnv(["KUBECONFIG=${KUBECONFIG}"]) {
                            // Get the latest image tag from the GIT_COMMIT environment variable
                            def imageTag = "1.0.${BUILD_NUMBER}"
                            // Show file in the directory
                            sh "ls -l ${manifest}"
                            
                            // Replace the placeholder ${IMAGE_TAG} in deployment.yaml with the actual image tag
                            echo "${env.pre_build}"
                            echo "${imageTag}"
                            sh """#!/bin/bash
                               sed 's|\${env.pre_build}|${imageTag}|' ${manifest}
                               cp -p ${manifest} ${manifest}_copy
                               grep 'image:' ${manifest}
                            """
                            
                             
                            // Apply deployment.yaml to the K8s cluster
                            //sh "kubectl apply -f deployment.yaml"
                            
                        }
                    }
                }
        }

    }
}
