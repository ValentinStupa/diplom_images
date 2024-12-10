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
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '10'))
        timestamps()    // Приписывает timestamp к шагам
    }
    stages {
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
                    def PREVIOUS_BUILD_NUMBER = sh(returnStdout: true, script: 'grep "image:" ${manifest}|awk -F \":\" \'{print $3}\'')
                    echo "Output: ${PREVIOUS_BUILD_NUMBER}"

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
                            //def manifest = "manifest_files/myapp/nginx_deploy.yml"

                            sh "echo 1.0.${BUILD_NUMBER}"
                            
                            // Show file in the directory
                            sh "ls -l ${manifest}"
                            
                            // Replace the placeholder ${IMAGE_TAG} in deployment.yaml with the actual image tag
                            sh "echo ${{PREVIOUS_BUILD_NUMBER}}"
                            sh "echo ${imageTag}"
                            sh "sed -i 's|\${PREVIOUS_BUILD_NUMBER}|\${imageTag}|' ${manifest}"
                            sh "grep 'image:' ${manifest}"
                             
                            // Apply deployment.yaml to the K8s cluster
                            //sh "kubectl apply -f deployment.yaml"
                            
                        }
                    }
                }
        }
    }
}
