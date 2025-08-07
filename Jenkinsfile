pipeline {
    agent any

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
                    docker run --rm \
                        -v "$PWD/app:/src" \
                        returntocorp/semgrep \
                        semgrep --config=auto \
                                --output=/src/../reports/semgrep_report.txt
                '''
            }
        }

        stage('Display Semgrep Report') {
            steps {
                sh 'cat reports/semgrep_report.txt || echo "No report found."'
            }
        }

        stage('Start Application') {
            steps {
                dir('app') {
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
  mkdir -p reports
  docker run --rm --user root \
    --network host \
    -v $PWD:/zap/wrk \
    -v $PWD/reports:/zap/reports \
    -t owasp/zap2docker-stable zap-baseline.py \
    -t http://localhost:3000 \
    -r /zap/reports/zap_report.html
'''
    }
}
 

        stage('Display ZAP Report Summary') {
            steps {
                sh 'ls -l reports && echo "ZAP scan completed. Check zap_report.html in the reports directory."'
            }
        }
    }
}
