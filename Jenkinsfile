pipeline {
    agent any

    environment {
        APP_PORT = '3000'
        REPORT_DIR = 'reports'
        SEMGREP_IMAGE = 'returntocorp/semgrep'
        ZAP_IMAGE = 'owasp/zap2docker-stable'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Onkar-kumbhar/DevSecOps.git'
            }
        }

        stage('Run Semgrep') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                    docker run --rm -v $(pwd):/src ${SEMGREP_IMAGE} semgrep scan --config=auto --output=/src/${REPORT_DIR}/semgrep_report.txt
                '''
            }
        }

        stage('Start Application') {
            steps {
                sh '''
                    docker-compose down || true
                    docker-compose up -d
                    sleep 10
                '''
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                    docker run --rm --network host -v $(pwd):/zap/wrk -v $(pwd)/${REPORT_DIR}:/zap/reports ${ZAP_IMAGE} zap-baseline.py -t http://localhost:${APP_PORT} -r zap_report.html
                    cp ${REPORT_DIR}/zap_report.html ${REPORT_DIR}/zap_report.txt || true
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                script {
                    def semgrepReport = readFile("${REPORT_DIR}/semgrep_report.txt")
                    echo "=== Semgrep Report ==="
                    echo semgrepReport
                }
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                script {
                    def zapReportPath = "${REPORT_DIR}/zap_report.txt"
                    if (fileExists(zapReportPath)) {
                        def zapReport = readFile(zapReportPath)
                        echo "=== ZAP Report Summary ==="
                        echo zapReport.take(5000) // limit output for log readability
                    } else {
                        echo "ZAP report not found. Skipping summary."
                    }
                }
            }
        }
    }

    post {
        always {
            sh 'docker-compose down || true'
            archiveArtifacts artifacts: "${REPORT_DIR}/*", allowEmptyArchive: true
        }
    }
}
