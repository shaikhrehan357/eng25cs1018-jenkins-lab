pipeline {
    agent any
    options {
        timestamps()
        timeout(time: 15, unit: 'MINUTES')
    }
    triggers {
        pollSCM('H/2 * * * *')
    }
    environment {
        APP_NAME = 'usn-calculator'
    }
    stages {
        stage('Build') {
            steps {
                sh 'python3 -m py_compile calculator.py'
                echo 'Syntax check passed'
            }
        }
        stage('Test') {
            steps {
                sh 'python3 -m pytest -v --junitxml=test-results.xml'
            }
        }
        stage('Package') {
            steps {
                sh 'mkdir -p dist'
                sh 'tar -czf dist/${APP_NAME}-${BUILD_NUMBER}.tar.gz calculator.py'
                archiveArtifacts artifacts: 'dist/*.tar.gz', fingerprint: true
            }
        }
        stage('Deploy to staging') {
            steps {
                input message: 'Deploy to staging?', ok: 'Deploy'
                sh 'mkdir -p /tmp/usn-staging'
                sh 'cp calculator.py /tmp/usn-staging/'
                sh 'ls -l /tmp/usn-staging'
            }
        }
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'test-results.xml'
        }
        success {
            echo "Build ${env.BUILD_NUMBER} of ${env.APP_NAME} passed"
        }
        failure {
            echo "Build ${env.BUILD_NUMBER} failed - read the Console Output"
        }
    }
}
