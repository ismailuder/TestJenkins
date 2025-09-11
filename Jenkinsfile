pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token') // SonarQube token
        IMAGE_NAME = "testjenkins:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                // Dotnet build/test/sonarscanner direkt container içinde çalıştırılıyor
                sh '''
                    dotnet sonarscanner begin /k:"TestJenkins" /d:sonar.login=$SONARQUBE /s:src/TestJenkins/TestJenkins.csproj
                    dotnet build src/TestJenkins/TestJenkins.csproj -c Release
                    dotnet test src/TestJenkins/TestJenkins.csproj -c Release
                    dotnet sonarscanner end /d:sonar.login=$SONARQUBE
                '''
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                    minikube image load $IMAGE_NAME
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully! 🎉"
        }
        failure {
            echo "Pipeline failed. ❌"
        }
    }
}
