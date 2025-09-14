pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN') // Jenkins credential ID
        DOTNET_TOOLS_PATH = "$HOME/.dotnet/tools"
        DOTNET_CLI_HOME = "$HOME"
    }

    options {
        // Workspace'i her build başında temizle
        skipDefaultCheckout()
    }

    stages {
        stage('Checkout SCM') {
            steps {
                deleteDir() // Workspace temizliği
                git url: 'https://github.com/ismailuder/TestJenkins.git', branch: 'master'
            }
        }

        stage('Build, Test & SonarQube') {
            steps {
                withEnv(["PATH=${DOTNET_TOOLS_PATH}:$PATH", "DOTNET_CLI_HOME=${DOTNET_CLI_HOME}"]) {
                    script {
                        // Eski scanner ve .sonarqube cache'ini temizle
                        sh '''
                            rm -rf $HOME/.dotnet/tools/.store/dotnet-sonarscanner
                            rm -rf .sonarqube
                        '''

                        // Dotnet SonarScanner'ı yükle (10.3)
                        sh '''
                            dotnet tool install --tool-path $HOME/.dotnet/tools dotnet-sonarscanner --version 10.3.0
                        '''

                        // SonarQube analizi
                        sh '''
                            $HOME/.dotnet/tools/dotnet-sonarscanner begin /k:TestJenkins /d:sonar.login=$SONAR_TOKEN /d:sonar.host.url=http://sonarqube:9000
                            dotnet build TestJenkins/TestJenkins.csproj -c Release
                            dotnet test TestJenkins/TestJenkins.csproj -c Release
                            $HOME/.dotnet/tools/dotnet-sonarscanner end /d:sonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Docker Build & Deploy to Minikube') {
            steps {
                echo 'Docker Build & Deploy stage skipped in case of earlier failure'
            }
        }
    }

    post {
        success {
            echo 'Pipeline succeeded ✅'
        }
        failure {
            echo 'Pipeline failed ❌'
        }
    }
}
