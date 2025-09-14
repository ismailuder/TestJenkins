pipeline {
    agent {
        docker { image 'mcr.microsoft.com/dotnet/sdk:8.0' }
    }

    environment {
        SONARQUBE = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/ismailuder/TestJenkins.git'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                withSonarQubeEnv('MySonarQube') {
                    sh 'dotnet sonarscanner begin /k:"TestJenkins" /d:sonar.login=$SONARQUBE'
                    sh 'dotnet build src/TestJenkins/TestJenkins.csproj -c Release'
                    sh 'dotnet test src/TestJenkins/TestJenkins.csproj -c Release'
                    sh 'dotnet sonarscanner end /d:sonar.login=$SONARQUBE'
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            agent { label 'docker' } // bu adım için ayrı agent belirleyebilirsin
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

    post {
        success { echo "Pipeline completed successfully! 🎉" }
        failure { echo "Pipeline failed. ❌" }
    }
}
