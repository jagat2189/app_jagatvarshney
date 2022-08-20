pipeline {
    agent any
    tools {
        nodejs "nodejs"
    }
    environment {
        scannerHome = tool 'SonarQubeScanner'
        dockerImage = ''
        image = 'jagatvarshney/nagp-devops'
        username = 'jagatvarshney'
        registryCredential = 'dockercredential'
    }
    stages {
        stage('Cloning Git') {
            steps {
                echo "checkout branch: " + env.BRANCH_NAME
                checkout([$class: 'GitSCM', branches: [[name: '*/'+env.BRANCH_NAME]], extensions: [], userRemoteConfigs: [[url: 'https://github.com/jagat2189/app_jagatvarshney.git']]])
            }
        }
        stage('Building Docker Image') {
            when {
                branch "master"
            }
            steps {
                script {
                    dockerImage = docker.build image
                }
            }
        }
        stage('Sonarqube Analysis') {
            when {
                branch "master"
            }
            steps {
                withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Test case execution') {
            when {
                branch "master"
            }
            steps {
                bat "npm install jest-environment-jsdom"
                bat "npm install -g jest"
                bat "npm test"
            }
        }
        stage('Push Image To Registry') {
            when {
                branch "master"
            }
            steps {
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push("i-${username}-${BRANCH_NAME}-${BUILD_NUMBER}")
                        dockerImage.push("i-${username}-${BRANCH_NAME}-latest")
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
		    steps {
                echo "Initiating Kubernetes deployment"
		        bat "kubectl --kubeconfig=C:\Users\jagatvarshney\.kube\Config apply -f deployment.yaml"
		    }
		}
    }
}
