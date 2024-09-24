pipeline {
    agent {
        docker {
            image 'node:16' 
        }
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test' 
            }
        }
        stage('Security Scan') {
            steps {
                sh 'npm install -g snyk' 
                sh 'snyk test' 
            }
        }
    }
    post {
        always {
            junit '**/test-results.xml' 
        }
        failure {
            script {
                echo 'Critical vulnerabilities detected! Halting the pipeline.'
                currentBuild.result = 'FAILURE' 
            }
        }
    }
}
