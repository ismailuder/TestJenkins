pipeline {
    agent any

    environment {
        TEMP_TOOLS = "${WORKSPACE}/.temp_dotnet_tools"
        PATH = "${TEMP_TOOLS}:${env.PATH}"
        DOCKER_IMAGE = "testjenkins:latest"
        KUBE_DEPLOYMENT = "testjenkins-deployment"
        KUBE_SERVICE = "testjenkins-service"
        NAMESPACE = "default"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Prepare .NET SonarScanner') {
            steps {
                script {
                    sh 'rm -rf $TEMP_TOOLS && mkdir -p $TEMP_TOOLS'
                    sh 'dotnet tool install --tool-path $TEMP_TOOLS dotnet-sonarscanner --version 10.3.0'
                }
            }
        }

        stage('Build & Test & SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        $TEMP_TOOLS/dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                        dotnet build TestJenkins/TestJenkins.csproj -c Release
                        dotnet test TestJenkins/TestJenkins.csproj -c Release
                        $TEMP_TOOLS/dotnet-sonarscanner end /d:sonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t $DOCKER_IMAGE ."
            }
        }

        stage('Push to Local Kubernetes') {
            steps {
                sh '''
                    # Docker image lokalde hazır, Kubernetes deployment update
                    kubectl apply -f k8s/testjenkins-deployment.yaml
                    kubectl apply -f k8s/testjenkins-service.yaml
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline başarılı ✅"
        }
        failure {
            echo "Pipeline başarısız ❌"
        }
    }
}
