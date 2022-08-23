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
    options {
        timestamps()
        timeout(time: 1, unit: "HOURS")
    }
    stages {
        stage('Build Docker Image') {
            steps {
                echo "Building Docker Image.."
                script {
                    dockerImage = docker.build image
                }
            }
        }
        stage('Sonarqube Analysis') {
            when {
                branch "develop"
            }
            steps {
                withSonarQubeEnv('Test_Sonar') {
                    bat "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Test Case Execution') {
            when {
                branch "master"
            }
            steps {
                echo "Executing Test Cases.."
                bat "npm install jest-environment-jsdom"
                bat "npm install -g jest"
                bat "npm test"
            }
        }
        stage('Push Image To DockerHub') {
            steps {
                echo "Pushing Docker Image to Docker Hub..."
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential ) {
                        dockerImage.push("i-${username}-${BRANCH_NAME}-${BUILD_NUMBER}")
                        dockerImage.push("i-${username}-${BRANCH_NAME}-latest")
                    }
                }
                echo "Pushed tag i-${username}-${BRANCH_NAME}-${BUILD_NUMBER}"
                echo "Pushed tag i-${username}-${BRANCH_NAME}-latest"
            }
        }
        stage('Kubernetes Deployment') {
		    steps {
                script {
                    def deployment_yaml = readFile file: "deployment.yaml"
                    deployment_yaml = deployment_yaml.replaceAll("imagename", "${image}:i-${username}-${BRANCH_NAME}-${BUILD_NUMBER}")
                    writeFile file: "deployment.yaml", text: deployment_yaml
                }
                echo "Initiating Kubernetes deployment in cluster..."

                bat "kubectl apply -f namespace.yaml"
                bat "kubectl apply -f secret.yaml"
                bat "kubectl apply -f configmap.yaml"
		        bat "kubectl apply -f deployment.yaml"

                echo "Deployment Completed..."
		    }
		}
    }
}
