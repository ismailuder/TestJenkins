pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('sonar-token') // Jenkins Credentials içinde ekli token
        DOTNET_ROOT = "/usr/share/dotnet"
        PATH = "${DOTNET_ROOT}:/root/.dotnet/tools:${env.PATH}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout') {
            steps {
                git branch: 'master',
                    url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Install .NET Tools') {
            steps {
                sh '''
                mkdir -p .temp_dotnet_tools
                dotnet tool install --tool-path .temp_dotnet_tools dotnet-sonarscanner --version 10.3.0
                '''
            }
        }

        stage('Build & SonarQube Analysis') {
            steps {
                sh '''
                ./.temp_dotnet_tools/dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=${SONAR_TOKEN} /d:sonar.host.url=http://sonarqube:9000
                dotnet build TestJenkins/TestJenkins.csproj -c Release
                dotnet test TestJenkins/TestJenkins.csproj -c Release
                ./.temp_dotnet_tools/dotnet-sonarscanner end /d:sonar.login=${SONAR_TOKEN}
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t testjenkins:latest TestJenkins/
                '''
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                # Minikube Docker environment
                eval $(minikube -p minikube docker-env)
                # Kubernetes deployment apply
                kubectl apply -f k8s/deployment.yaml
                kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline başarıyla tamamlandı ✅"
        }
        failure {
            echo "Pipeline başarısız ❌"
        }
    }
}
