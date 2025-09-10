pipeline {
    agent any

    environment {
        SONARQUBE = credentials('test') // Jenkins'te eklediğin SonarQube token ID
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    docker.image('mcr.microsoft.com/dotnet/sdk:7.0').inside {
                        withSonarQubeEnv('SONARQUBE') {
                            sh 'dotnet sonarscanner begin /k:"TestJenkins" /d:sonar.login=$SONARQUBE /s:src/TestJenkins/TestJenkins.csproj'
                            sh 'dotnet build src/TestJenkins/TestJenkins.csproj'
                            sh 'dotnet sonarscanner end /d:sonar.login=$SONARQUBE'
                        }
                    }
                }
            }
        }

        stage('Build & Test') {
            steps {
                script {
                    docker.image('mcr.microsoft.com/dotnet/sdk:7.0').inside {
                        sh 'dotnet test src/TestJenkins/TestJenkins.csproj'
                    }
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh """
                # Build Docker image
                docker build -t testjenkins:latest .

                # Load image into Minikube
                minikube image load testjenkins:latest

                # Apply Kubernetes manifests
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}
