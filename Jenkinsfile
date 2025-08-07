pipeline {
    agent any

    environment {
        APP_PORT = "3000"
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
                mkdir -p reports
                docker run --rm -v $(pwd)/app:/src returntocorp/semgrep semgrep --config "p/owasp-top-ten" --json > reports/semgrep_report.json
                docker run --rm -v $(pwd)/app:/src returntocorp/semgrep semgrep --config "p/owasp-top-ten" --text > reports/semgrep_report.txt
                '''
            }
        }

        stage('Start Application') {
            steps {
                sh '''
                docker-compose down || true
                docker-compose up -d
                sleep 10  # wait for app to start
                '''
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

        stage('Display Semgrep Report') {
            steps {
                sh 'cat reports/semgrep_report.txt || echo "Semgrep report not found."'
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                sh 'cat reports/zap_report.txt || echo "ZAP report not found."'
            }
        }
    }

    post {
        always {
            echo "Pipeline completed. All reports are available in the 'reports' directory."
        }
    }
}
