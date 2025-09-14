pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token')  // Jenkins içinde oluşturduğun token
        IMAGE_NAME = "testjenkins:latest"
        KUBE_CONFIG = "/home/jenkins/.kube/config"
        DOTNET_ROOT = "/usr/share/dotnet"
        PATH = "/usr/share/dotnet:/root/.dotnet/tools:$PATH"
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
                    echo "Using dotnet from: $(which dotnet)"
                    dotnet --version

                    dotnet sonarscanner begin /k:"TestJenkins" /d:sonar.login=$SONARQUBE /d:sonar.host.url=http://sonarqube:9000
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
