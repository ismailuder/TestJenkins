pipeline {
    agent any

    environment {
        TEMP_TOOLS = "${WORKSPACE}/.temp_dotnet_tools"
        PATH = "${TEMP_TOOLS}:${env.PATH}"
    }

    stages {
        stage('Checkout SCM') {
			steps {
				git url: 'https://github.com/ismailuder/TestJenkins.git', branch: 'main'
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

        stage('Build Docker Image') {
            steps {
                echo 'Docker image build ediliyor...'
                sh 'docker build -t testjenkins:latest -f TestJenkins/Dockerfile TestJenkins'
            }
        }

        // Opsiyonel: Docker Hub’a push etmek istersen bu kısmı açabilirsin
        /*
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker tag testjenkins:latest your-dockerhub-username/testjenkins:latest
                        docker push your-dockerhub-username/testjenkins:latest
                    '''
                }
            }
        }
        */

        stage('Kubernetes Node Check') {
            steps {
                echo 'Kubernetes Node Durumu Kontrol Ediliyor...'
                sh 'kubectl get nodes'
            }
        }

        stage('Deploy to Prod') {
            steps {
                echo 'Prod ortamına deploy ediliyor...'
                sh 'kubectl apply -f TestJenkins/k8s/deployment.yaml -n prod'
            }
        }
    }

    post {
        success { echo "Pipeline başarılı ✅" }
        failure { echo "Pipeline başarısız ❌" }
    }
}
