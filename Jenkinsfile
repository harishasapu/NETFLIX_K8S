pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/harishasapu/NETFLIX_K8S.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('SonarQube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Netflix \
                    -Dsonar.projectKey=Netflix '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-cred'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker'){   
                       sh "docker build --build-arg TMDB_V3_API_KEY=7298fa10b3dcc456768039a9 -t netflix ."
                       sh "docker tag netflix harishasapu/netflix:latest "
                       sh "docker push harishasapu/netflix:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                 sh " trivy image --format table harishasapu/netflix:latest "
            }
        }
        stage('Deploy to container'){
            steps{
                 sh 'docker run -d --name netflix -p 9091:80 harishasapu/netflix:latest'
            }
        }
        stage('Deploy to kubernets'){
            steps{
                script{
                    dir('Kubernetes'){
                        withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s-cred', namespace: '', restrictKubeConfigAccess: false, serverUrl: ''){
                             sh 'kubectl apply -f deployment.yml'
                             sh 'kubectl apply -f service.yml'
                        }
                    }
                }
            }
        }
    }
}

 
