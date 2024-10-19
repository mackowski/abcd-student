pipeline {
    agent any
    options {
        skipDefaultCheckout(true)
    }
    stages {
        stage('Code checkout from GitHub') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'github', url: 'https://github.com/mackowski/abcd-student', branch: 'main'
                }
            }
        }
        stage('Example') {
            steps {
                echo 'Hello!'
                sh 'ls -la'
            }
        }

        stage('[trufflehog] Secrets Scan') {
            steps {
                sh 'trufflehog git file://. --branch=main --json > ${WORKSPACE}/secrets-trufflehog-scanner.json'
            }
            post {
                always {
                    defectDojoPublisher(artifact: 'secrets-trufflehog-scanner.json', 
                        productName: 'Juice Shop', 
                        scanType: 'Trufflehog Scan', 
                        engagementName: 'jakub.mackowski@relativity.com')
                }
            }
        }
        
        stage('[OSV] SCA Scan') {
            steps {
                sh 'osv-scanner scan --lockfile package-lock.json --format json --output ${WORKSPACE}/sca-osv-scanner.json' || true
            }
            post {
                always {
                    defectDojoPublisher(artifact: 'sca-osv-scanner.json', 
                        productName: 'Juice Shop', 
                        scanType: 'OSV Scan', 
                        engagementName: 'jakub.mackowski@relativity.com')
                }
            }
        }

    }
}
