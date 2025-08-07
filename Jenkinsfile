pipeline {
    agent any

    environment {
        APP_PORT = "3000"
        REPORT_DIR = "reports"
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
                    pip install semgrep
                    mkdir -p ${REPORT_DIR}
                    semgrep --config=auto . --json > ${REPORT_DIR}/semgrep_report.json || true
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                sh '''
                    echo "------ Semgrep Report Summary ------"
                    if [ -f ${REPORT_DIR}/semgrep_report.json ]; then
                        cat ${REPORT_DIR}/semgrep_report.json | jq '.results | length'
                    else
                        echo "Semgrep report not found!"
                    fi
                '''
            }
        }

        stage('Start Application') {
            steps {
                sh 'docker-compose up -d'
            }
        }

        stage('Run ZAP Scan') {
            steps {
                sh '''
                    mkdir -p ${REPORT_DIR}
                    docker run --rm \
                        -v $PWD/${REPORT_DIR}:/zap/reports \
                        -v $PWD:/zap/wrk \
                        -t owasp/zap2docker-stable \
                        zap-baseline.py \
                        -t http://host.docker.internal:${APP_PORT} \
                        -r zap_report.html || true
                '''
            }
        }

        stage('Display ZAP Report Summary') {
            steps {
                sh '''
                    echo "------ ZAP Report Summary ------"
                    if [ -f ${REPORT_DIR}/zap_report.html ]; then
                        grep -A 5 "Risk Level" ${REPORT_DIR}/zap_report.html || echo "Could not parse risk levels."
                    else
                        echo "ZAP report not found!"
                    fi
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline completed. All reports are available in the '${REPORT_DIR}' directory."
        }
    }
}
