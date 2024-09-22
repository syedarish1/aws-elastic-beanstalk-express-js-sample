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
                sh 'npm install -g snyk' // Install Snyk globally
                sh 'snyk test' // Run Snyk vulnerability scan
            }
        }
    }
    post {
        always {
            junit '**/test-results.xml' // Always capture test results
        }
        failure {
            script {
                echo 'Critical vulnerabilities detected! Halting the pipeline.' 
                currentBuild.result = 'FAILURE' // Fail the build if vulnerabilities are found
            }
        }
    }
}
