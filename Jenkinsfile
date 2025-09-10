pipeline {
    agent any

    environment {
        SONARQUBE = credentials('test')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                sh '''
                    docker run --rm -v $PWD:/app -w /app mcr.microsoft.com/dotnet/sdk:8.0 bash -c "
                        dotnet sonarscanner begin /k:'TestJenkins' /d:sonar.login=$SONARQUBE /s:src/TestJenkins/TestJenkins.csproj &&
                        dotnet build src/TestJenkins/TestJenkins.csproj &&
                        dotnet test src/TestJenkins/TestJenkins.csproj &&
                        dotnet sonarscanner end /d:sonar.login=$SONARQUBE
                    "
                '''
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh '''
                    docker build -t testjenkins:latest .
                    minikube image load testjenkins:latest
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }
}
