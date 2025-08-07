pipeline {
    agent any

    environment {
        REPORTS_DIR = "${WORKSPACE}/reports"
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
                    docker run --rm -v "$PWD/app:/src" returntocorp/semgrep semgrep --config=auto --output=/src/../reports/semgrep_report.txt
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                sh 'cat reports/semgrep_report.txt || echo "No Semgrep report found."'
            }
        }

        stage('Start Application') {
            steps {
                dir('app') {
                    sh '''
                        nohup python3 -m http.server 3000 &
                        sleep 5
                    '''
                }
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p reports zap-wrk
                    docker run --rm \
                        -v "$PWD/reports:/zap/reports" \
                        -v "$PWD/zap-wrk:/zap/wrk" \
                        -t owasp/zap2docker-stable \
                        zap-baseline.py -t http://host.docker.internal:3000 -r zap_report.html
                '''
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                sh 'cat reports/zap_report.html || echo "ZAP report not found."'
            }
        }
    }
}
