pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token')  // Jenkins içinde oluşturduğun SonarQube token
        KUBE_CONFIG = "/home/jenkins/.kube/config"
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
                    # .NET SDK zaten container içinde mevcut, Docker run gerek yok
                    dotnet sonarscanner begin /k:'TestJenkins' /d:sonar.login=$SONARQUBE /d:sonar.host.url=http://sonarqube:9000
                    dotnet build src/TestJenkins/TestJenkins.csproj -c Release
                    dotnet test src/TestJenkins/TestJenkins.csproj -c Release
                    dotnet sonarscanner end /d:sonar.login=$SONARQUBE
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    # Docker build gerekirse minikube image load ile yükle
                    docker build -t testjenkins:latest .
                    minikube image load testjenkins:latest
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
