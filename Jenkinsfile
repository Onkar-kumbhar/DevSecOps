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
                    docker run --rm -v "$PWD:/src" returntocorp/semgrep semgrep scan --config=auto --json > reports/semgrep_report.json
                    cat reports/semgrep_report.json > reports/semgrep_report.txt
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                sh 'cat reports/semgrep_report.txt'
            }
        }

        stage('Start Application') {
            steps {
                sh 'docker-compose up -d'
                sleep(time: 10, unit: "SECONDS")  // Wait for app to start
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p reports
                    docker run --rm --user root --network host \
                        -v "$PWD:/zap/wrk" -v "$PWD/reports:/zap/reports" \
                        owasp/zap2docker-stable zap-baseline.py -t http://localhost:$APP_PORT > reports/zap_report.txt
                '''
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                sh 'cat reports/zap_report.txt'
            }
        }
    }

    post {
        always {
            echo "Pipeline completed. All reports are available in the 'reports' directory."
        }
    }
}
