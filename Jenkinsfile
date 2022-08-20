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
                bat "npm install jest-environment-jsdom"
                bat "npm install -g jest"
                bat "npm test"
            }
        }
        stage('Push Image To DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential ) {
                        dockerImage.push("i-${username}-${BRANCH_NAME}-${BUILD_NUMBER}")
                        dockerImage.push("i-${username}-${BRANCH_NAME}-latest")
                    }
                }
            }
        }
        stage('Kubernetes Deployment') {
		    steps {
                script {
                    def deployment_yaml = readFile file: "deployment.yaml"
                    deployment_yaml = deployment_yaml.replaceAll("imagename", "${image}:i-${username}-${BRANCH_NAME}-${BUILD_NUMBER}")
                    writeFile file: "deployment.yaml", text: deployment_yaml
                }
                echo "Initiating Kubernetes deployment"
		        bat "kubectl --kubeconfig=C:\\Users\\jagatvarshney\\.kube\\Config apply -f deployment.yaml"
		    }
		}
    }
}
