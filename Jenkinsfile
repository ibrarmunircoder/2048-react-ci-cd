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
        
      
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Game \
                    -Dsonar.projectKey=Game '''
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

        stage('Build and Push docker image') {
            steps {
                script {
                     withCredentials([
        usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'USER', passwordVariable: 'PASS')
        ]) {
            sh 'docker build -t ibrarmunir009/my-repo:game-1.0 .'
            sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
            sh 'docker push ibrarmunir009/my-repo:game-1.0'
    }
                }
            }
        }

         stage("TRIVY"){
            steps{
                sh "trivy image sevenajay/2048:latest > trivy.txt" 
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name 2048 -p 3000:3000 ibrarmunir009/my-repo:game-1.0'
            }
        }
    }
}
