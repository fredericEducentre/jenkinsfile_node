pipeline {
    agent { label 'angular-sonnarscanner' }
    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                git branch: 'main', credentialsId: "${credentials}", url: "${url_repo_github}"
                sh "git remote set-url origin https://${username}:${git_pat}@github.com/${username}/${repo_name}.git"
                sh '''
                    last_pull_request=$(git ls-remote origin 'pull/*/head' | tail -n 1 )
                    last_number=$(echo "$last_pull_request" | grep -oE '/[0-9]+/' | tail -1 | tr -d '/' )
                    git fetch origin "pull/$last_number/head:pull_$last_number"
                    git checkout "pull_$last_number"
                '''
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