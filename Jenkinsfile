pipeline {
    agent any
    environment {
        imageName = "fleetman-queue"
        registryCredentials = "nexus"
        registry = "nexus-registry.eastus.cloudapp.azure.com:8085/"
        dockerImage = ''
    }

    stages {

        stage('Git Preparation') {
            steps {
                cleanWs()
                checkout scm
                sh 'git rev-parse --short HEAD > .git/commit-id'
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
                sh 'chmod 775 *'
            }
        }
        stage('Build Docker image') {
            steps {
                 script {
                    dockerImage = docker.build imageName + ":${commit_id}"
                 }
            }
        }

        stage('Push Docker image to Nexus Registry') {
            steps {
                script {
                    docker.withRegistry( 'http://'+registry, registryCredentials) {
                        dockerImage.push()
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('Trigger K8S Manifest Updating') {
            steps {
                build job: 'k8s-update-manifests-fleetman-queue', parameters: [string(name: 'DOCKERTAG', value: commit_id)]
            }
        }
    }
}
