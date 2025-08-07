pipeline {
    agent any

    environment {
        APP_PORT = "3000"
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
                    mkdir -p reports
                    docker run --rm -v "$PWD/app:/src" returntocorp/semgrep \
                        semgrep --config=auto --output=/src/../reports/semgrep_report.txt || true
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                script {
                    echo "=== Semgrep Report ==="
                    if (fileExists('reports/semgrep_report.txt')) {
                        sh 'cat reports/semgrep_report.txt'
                    } else {
                        echo "Semgrep report not found."
                    }
                }
            }
        }

        stage('Start Application') {
            steps {
                dir('app') {
                    sh '''
                        nohup python3 -m http.server ${APP_PORT} &
                        sleep 5
                    '''
                }
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p reports
                    (
                      docker run --rm --user root --network host \
                        -v "$PWD:/zap/wrk" \
                        -v "$PWD/reports:/zap/reports" \
                        owasp/zap2docker-stable zap-baseline.py -t http://localhost:$APP_PORT
                    ) > reports/zap_report.txt || true
                '''
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                script {
                    echo "=== ZAP Report Summary ==="
                    if (fileExists('reports/zap_report.txt')) {
                        sh 'cat reports/zap_report.txt'
                    } else {
                        echo "ZAP report not found."
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
