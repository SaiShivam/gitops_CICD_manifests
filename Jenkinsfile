pipeline{
    agent any
    environment{
        DOCKERHUB_USERNAME = "saishivam"
        APP_NAME = "gitops-argo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = "dockerhub"
    }

    stages{
        stage('cleanup workspace'){
            steps{
                script{
                    cleanWs()
                }
            }

        }

        stage("checkout scm"){
            steps{
                script{
                    git credentialsId: 'github',
                    url: 'https://github.com/SaiShivam/gitops_CICD.git',
                    branch: 'main'


                }
            }
        }

        stage("build docker image"){
            steps{
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }

        stage("push docker image"){
            steps{
                script{
                    docker.withRegistry('', REGISTRY_CREDS){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push("latest")
                    }
                 
                }
            }

        }

        stage("delete docker images"){
            steps{
                script{
                    sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }

        }

        stage("updating kubernetes deployment file"){
            steps{
                script{
                    sh """
                    cat deployment.yml
                    sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' deployment.yml
                    cat deployment.yml
                    """
                }
            }

        }

        stage("push the changed deployment file to git"){
            steps{
                script{
                    sh """
                    git config --global user.name "saishivam"
                    git config --global user.email "saishivam538@outlook.com"
                    git add deployment.yml
                    git commit -m "updated deployment file"
                    """
                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        
                        sh "git push https://github.com/SaiShivam/gitops_CICD.git main"

                            }
                }
            }

        }





    }
}