pipeline {
    agent any

    environment {
        SONARQUBE = credentials('sonar-token')
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
                sh '''
                    docker run --rm \
                        --network dev-network \
                        -v $PWD:/app -w /app mcr.microsoft.com/dotnet/sdk:8.0 bash -c "
                            dotnet sonarscanner begin /k:'TestJenkins' /d:sonar.login=$SONARQUBE /d:sonar.host.url=http://sonarqube:9000 &&
                            dotnet build src/TestJenkins/TestJenkins.csproj -c Release &&
                            dotnet test src/TestJenkins/TestJenkins.csproj -c Release &&
                            dotnet sonarscanner end /d:sonar.login=$SONARQUBE
                        "
                '''
            }
        }

        stage('Docker Build & Minikube Deploy') {
            steps {
                sh '''
                    # Jenkins container içinden host Docker ve Minikube kullanımı
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
            echo "Pipeline başarıyla tamamlandı 🎉"
        }
        failure {
            echo "Pipeline başarısız ❌"
        }
    }
}
