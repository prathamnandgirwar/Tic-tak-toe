```groovy

pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'nodejs16'
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
                git branch: 'main', url: 'https://github.com/prathamnandgirwar/Youtube-clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=tic-tak-toe \
                    -Dsonar.projectKey=tic-tak-toe '''
                }
            }
        }
        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
         stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'p', usernameVariable: 'u')]) {
    // some block
                    sh "docker login -u ${env.u} -p ${env.p}"
                       sh "docker build -t yic-tak-toe ."
                       sh "docker tag tic-tak-toe prathamnandgiwar/tic-tak-toe:latest "
                       sh "docker push prathamnandgiwar/tic-tak-toe:latest "
                    
                }
                 
                }
            
        }
        stage("TRIVY"){
            steps{
                sh "trivy image prathamnandgiwar/tic-tak-toe:latest > trivyimage.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name tic-tak-toe -p 3000:3000 prathamnandgiwar/tic-tak-toe:latest'
            }
        }
        
    }
}

```
