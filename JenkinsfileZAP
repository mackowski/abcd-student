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
        stage('[ZAP] Baseline passive-scan') {
            steps {
                sh '''
                    docker run --name juice-shop-ci -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap-ci --add-host=host.docker.internal:host-gateway -v /Users/jakub.mackowski/ABCDevSecOps/abcd-student/.zap:/zap/wrk/:rw -t ghcr.io/zaproxy/zaproxy:stable bash -c "zap.sh -cmd -addonupdate; zap.sh -cmd -addoninstall communityScripts -addoninstall pscanrulesAlpha -addoninstall pscanrulesBeta -autorun /zap/wrk/passive.yaml" || true
                '''
            }
            post {
                always {
                    sh 'docker cp zap-ci:/zap/wrk/reports/zap_html_report.html ${WORKSPACE}/zap_html_report.html'
                    sh 'docker cp zap-ci:/zap/wrk/reports/zap_xml_report.xml ${WORKSPACE}/zap_xml_report.xml'
                    sh 'docker stop zap-ci juice-shop-ci'
                    
                    defectDojoPublisher(artifact: 'zap_xml_report.xml', 
                    productName: 'Juice Shop', 
                    scanType: 'ZAP Scan', 
                    engagementName: 'jakub.mackowski@relativity.com')
                }
            }
        }

    }
}
