pipeline {
    agent any

    environment {
        TEMP_TOOLS = "${WORKSPACE}/.temp_dotnet_tools"
        PATH = "${TEMP_TOOLS}:${env.PATH}"
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Prepare SonarScanner') {
            steps {
                script {
                    sh 'rm -rf $TEMP_TOOLS && mkdir -p $TEMP_TOOLS'
                    sh 'dotnet tool install --tool-path $TEMP_TOOLS dotnet-sonarscanner --version 10.3.0'
                }
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                withCredentials([string(credentialsId: 'SONAR_TOKEN', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                        dotnet build TestJenkins/TestJenkins.csproj -c Release
                        dotnet test TestJenkins/TestJenkins.csproj -c Release
                        dotnet-sonarscanner end /d:sonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    # Docker image build
                    docker build -t testjenkins:latest -f TestJenkins/Dockerfile TestJenkins

                    # Namespace oluştur
                    kubectl create namespace prod --dry-run=client -o yaml | kubectl apply -f -

                    # Deployment ve Service apply
                    kubectl apply -f TestJenkins/k8s/deployment.yaml -n prod
                    kubectl apply -f TestJenkins/k8s/service.yaml -n prod

                    # Pod ve Service kontrol
                    kubectl get pods -n prod
                    kubectl get svc -n prod
                '''
            }
        }
    }

    post {
        success { echo "Pipeline başarılı ✅" }
        failure { echo "Pipeline başarısız ❌" }
    }
}
