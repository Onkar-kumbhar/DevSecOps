pipeline {
    agent any

    environment {
        APP_DIR = 'app'
        REPORT_DIR = 'reports'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git 'https://github.com/Onkar-kumbhar/DevSecOps.git'
            }
        }

        stage('Run Semgrep') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                    docker run --rm -v "$PWD/${APP_DIR}:/src" returntocorp/semgrep semgrep --config=auto --output=/src/../${REPORT_DIR}/semgrep_report.txt
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                sh 'cat ${REPORT_DIR}/semgrep_report.txt'
            }
        }

        stage('Start Application') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        nohup python3 -m http.server 3000 > /dev/null 2>&1 &
                        sleep 5
                    '''
                }
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                    docker run --rm --user root --network host \
                    -v "$PWD:/zap/wrk" \
                    -v "$PWD/${REPORT_DIR}:/zap/reports" \
                    owasp/zap2docker-stable zap-baseline.py \
                    -t http://localhost:3000 \
                    > ${REPORT_DIR}/zap_report.txt
                '''
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                script {
                    def zapReport = "${REPORT_DIR}/zap_report.txt"
                    if (fileExists(zapReport)) {
                        echo "ZAP Report Summary:"
                        sh "cat ${zapReport}"
                    } else {
                        error "ZAP report not found!"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
    }
}
