pipeline {
    agent any
    environment {
        PROJECT_ID = 'ethereal-art-313711'
        LOCATION = 'europe-central2-a'
        CREDENTIALS_ID = 'gke'
        CLUSTER_NAME_TEST = 'stage'
        CLUSTER_NAME_PROD = 'prod'
	DOCKER_HUB_USERNAME = 'rococoshka'          
    }
    stages {
        stage("Checkout code") {
            steps {
                checkout scm
            }
        }
        stage("Build image") {
            steps {
                script {
                    myapp = docker.build("${env.DOCKER_HUB_USERNAME}/hello:${env.BUILD_ID}")
                }
            }
        }
        stage("Push image") {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                            myapp.push("latest")
                            myapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }       
        stage('Deploy to GKE test cluster') {
            steps{
                sh "sed -i 's/hello:latest/hello:${env.BUILD_ID}/g' deployment.yaml"
                step([$class: 'KubernetesEngineBuilder', projectId: "${env.PROJECT_ID}", clusterName: env.CLUSTER_NAME_TEST, zone: "${env.LOCATION}", manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }
        stage('Deploy to GKE production cluster') {
            steps{
                input message:"Proceed with final deployment?"
                step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME_PROD, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            }
        }   
    }    
}
