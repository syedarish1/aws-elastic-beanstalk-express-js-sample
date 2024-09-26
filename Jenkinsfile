pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }
    stages {
        stage('Dependencies Installation') {
            steps {
                sh 'npm install --save'
                echo 'Installing Finished'

                sh 'npm install snyk --save-dev'
                echo 'Snyk Installation completed'

                withCredentials([string(credentialsId: 'snyk_token', variable: 'SNYK_TOKEN')]) {
                    sh './node_modules/.bin/snyk auth $SNYK_TOKEN'
                    echo 'Snyk Authentication Completed'
                }
            }
        }
        stage('Building Phase') {
            steps {
                sh 'npm install --save'
                echo 'Build Phase Completed'
            }
        }

        stage('Snyk Security Scan Phase') {
            steps {
                script {
                    def snykResults = sh(script: './node_modules/.bin/snyk test --json', returnStdout: true)
                    def jsonResults = readJSON(text: snykResults)
                    if (jsonResults.vulnerabilities.any { it.severity == 'critical' }) {
                        error("Vulnerabilities found! Check snyk-report.json.")
                    } else {
                        writeFile file: 'snyk-report.json', text: snykResults
                    }
                }
                echo 'Security Scan Completed'
            }
            post {
                success {
                    echo 'Security Scan passed!'
                }
                failure {
                    echo 'Failed due to vulnerabilities.'
                }
            }
        }

        stage('Test Phase') {
            steps {
                script {
                    if (fileExists('package.json')) {
                        def pkg = readJSON file: 'package.json'
                        if (pkg.scripts && pkg.scripts.test) {
                            sh 'npm test'
                            echo 'Tests completed.'
                        } else {
                            echo 'No test script found in package.json. Skipping Test Phase.'
                        }
                    } else {
                        echo 'package.json not found. Skipping Test Phase.'
                    }
                }
            }
            post {
                success {
                    echo 'Tests passed!'
                }
                failure {
                    echo 'Some tests failed. Check logs for details.'
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline Completed.'
        }
    }
}
