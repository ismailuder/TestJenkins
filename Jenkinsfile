pipeline {
    agent any
    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')  // Jenkins Credentials
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Prepare SonarScanner') {
            steps {
                sh '''
                    mkdir -p .temp_dotnet_tools
                    dotnet tool install --tool-path .temp_dotnet_tools dotnet-sonarscanner --version 10.3.0
                '''
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                sh '''
                    ./.temp_dotnet_tools/dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                    dotnet build TestJenkins/TestJenkins.csproj -c Release
                    dotnet test TestJenkins/TestJenkins.csproj -c Release
                    ./.temp_dotnet_tools/dotnet-sonarscanner end /d:sonar.login=$SONAR_TOKEN
                '''
            }
        }

        stage('Docker Build') {
            steps {
                dir('TestJenkins') {
                    sh 'docker build -t testjenkins:latest .'
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    # Minikube docker environment kullan
                    eval $(minikube -p minikube docker-env)

                    # Image deploy için
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }
    }

    post {
        failure {
            echo "Pipeline başarısız ❌"
        }
        success {
            echo "Pipeline başarıyla tamamlandı ✅"
        }
    }
}
