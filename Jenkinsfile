pipeline {
    agent any

    environment {
        APP_PORT = '3000'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/Onkar-kumbhar/DevSecOps.git', branch: 'main'
            }
        }

        stage('Run Semgrep') {
            steps {
                sh '''
                    mkdir -p reports
                    docker run --rm -v "$PWD/app:/src" returntocorp/semgrep \
                    semgrep --config=auto --output=/src/../reports/semgrep_report.txt
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                script {
                    def semgrepReport = 'reports/semgrep_report.txt'
                    if (fileExists(semgrepReport)) {
                        echo "=== Semgrep Report ==="
                        sh "cat ${semgrepReport}"
                    } else {
                        error "Semgrep report not found!"
                    }
                }
            }
        }

        stage('Start Application') {
            steps {
                dir('app') {
                    sh '''
                        nohup python3 -m http.server $APP_PORT > /dev/null 2>&1 &
                        sleep 5
                    '''
                }
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p reports
                    docker run --rm --user root --network host \
                    -v "$PWD:/zap/wrk" \
                    -v "$PWD/reports:/zap/reports" \
                    owasp/zap2docker-stable zap-baseline.py \
                    -t http://localhost:$APP_PORT \
                    > reports/zap_report.txt
                '''
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                script {
                    def zapReport = 'reports/zap_report.txt'
                    if (fileExists(zapReport)) {
                        echo "=== ZAP Report ==="
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
            echo "Pipeline completed. All reports are available in the 'reports' directory."
        }
    }
}

