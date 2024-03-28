pipeline {
    agent { label 'angular-sonnarscanner' }
    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git branch: 'main', credentialsId: "${credentials}", url: "${url_repo_github}"
            }
        }
        stage('Build') {
            steps {
                sh '''
                    npm install
                    npx tsc index.ts --outDir dist
                '''
            }
        }
        stage('Test') {
            steps {
                sh 'npm test'
                withSonarQubeEnv('sonar-srv-1') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=${projectKey} \
                          -Dsonar.sources=. \
                          -Dsonar.host.url=${ip_adress} \
                          -Dsonar.token=${token}
                    """
                }
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
