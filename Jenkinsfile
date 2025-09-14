pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token')  // Jenkins içinde oluşturduğun SonarQube token
        IMAGE_NAME = "testjenkins:latest"
        KUBE_CONFIG = "/home/jenkins/.kube/config"
        DOTNET_CLI_HOME = "/var/jenkins_home"  // SonarScanner için gerekli
        PATH = "$PATH:/var/jenkins_home/.dotnet/tools:/usr/share/dotnet"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                sh """
                    # SonarScanner ve dotnet path ayarları
                    export PATH=\$PATH:/var/jenkins_home/.dotnet/tools:/usr/share/dotnet
                    export DOTNET_CLI_HOME=/var/jenkins_home

                    # SonarQube analizi başlat
                    dotnet sonarscanner begin /k:TestJenkins /d:sonar.login="${SONARQUBE}" /d:sonar.host.url="http://sonarqube:9000"

                    # Build ve test
                    dotnet build src/TestJenkins/TestJenkins.csproj -c Release
                    dotnet test src/TestJenkins/TestJenkins.csproj -c Release

                    # SonarQube analizi bitir
                    dotnet sonarscanner end /d:sonar.login="${SONARQUBE}"
                """
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                sh """
                    docker build -t $IMAGE_NAME .
                    minikube image load $IMAGE_NAME
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                """
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
