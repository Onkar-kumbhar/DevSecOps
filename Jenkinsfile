pipeline {
    agent any

    environment {
        APP_PORT = '3000'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/Onkar-kumbhar/DevSecOps.git'
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

        stage('Run Semgrep Scan') {
            steps {
                sh '''
                    mkdir -p reports
                    docker run --rm -v "$PWD:/src" returntocorp/semgrep semgrep --config=auto /src > reports/semgrep_report.txt
                '''
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
                        -r zap_report.html > reports/zap_report.txt
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: 'reports/**', allowEmptyArchive: true
            }
        }

    }

    post {
        always {
            echo 'Cleaning up...'
            sh 'docker-compose down || true'
        }
    }
}
