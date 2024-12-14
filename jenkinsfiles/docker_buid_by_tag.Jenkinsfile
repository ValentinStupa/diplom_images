#!groovy

/* groovylint-disable-next-line CompileStatic */
properties([disableConcurrentBuilds()])     // Не позволяет одновременно запускать несколько сборок

pipeline {
    environment {
        registry = 'valentinstupa/custom-nginx'
        registryCredential = 'dockerhub_access'
        dockerImage = ''
        KUBECONFIG = "/var/lib/jenkins/.kube/config"
        manifest = "manifest_files/myapp/nginx_deploy.yml"
        version = 'v'
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
        stage("Get last git tag") {
            steps {
                script {
                    def git_tag = sh(returnStdout: true, script: "git tag --points-at HEAD").trim()
                    echo "Git tag: $git_tag"
                    env.tag = git_tag
                } 
            }
        }
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
                        sed -i -e '\$a<tagname>${env.tag}</tagname>' index.html
                        """                        
                }   
            }
        }
        stage('Building image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":${env.tag}"
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
                sh "docker rmi $registry:${env.tag}"
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
                            // Get the latest image tag from the GIT_COMMIT environment variable
                            def imageTag = "${env.tag}"
                            
                            // Other option to replace tag into files

                            // def file = readFile manifest
                            // file = file.replace(env.pre_build, imageTag)
                            // writeFile file: manifest, text: file
                            
                            // Show file in the directory
                            sh "ls -l ${manifest}"
                            
                            // Replace the placeholder ${IMAGE_TAG} in deployment.yaml with the actual image tag
                            sh """
                               sed -i 's|${env.pre_build.trim()}|${imageTag}|' ${manifest}
                               grep 'image:' ${manifest}
                            """                        
                    }
                }
        }
        stage('Deploy to K8s cluster') {
            when {tag pattern: "${env.tag}"}
            steps {
                script {
                    // Set KUBECONFIG environment variable
                    withEnv(["KUBECONFIG=${KUBECONFIG}"]){
                            
                            // Apply manifest.yaml to the K8s cluster
                            
                            sh "kubectl get nodes"
                            // sh """
                            //    kubectl get nodes
                            //    kubectl apply -f ${manifest}
                            //    kubectl describe deployment.apps/nginx | grep 'Image:'
                            // """

                        }
                }

            }
        }
    }
}
