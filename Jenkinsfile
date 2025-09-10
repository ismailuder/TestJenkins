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

        stage('SonarQube Analysis & Build & Test') {
            steps {
                script {
                    // Tek bir .NET SDK image kullanıyoruz
                    docker.image('mcr.microsoft.com/dotnet/sdk:8.0').inside {
                        withSonarQubeEnv('SONARQUBE') {
                            sh 'dotnet sonarscanner begin /k:"TestJenkins" /d:sonar.login=$SONARQUBE /s:src/TestJenkins/TestJenkins.csproj'
                            sh 'dotnet build src/TestJenkins/TestJenkins.csproj'
                            sh 'dotnet test src/TestJenkins/TestJenkins.csproj'
                            sh 'dotnet sonarscanner end /d:sonar.login=$SONARQUBE'
                        }
                    }
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh """
                docker build -t testjenkins:latest .
                minikube image load testjenkins:latest
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                """
            }
        }
    }
}
