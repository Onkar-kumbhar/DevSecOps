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
    }
}
