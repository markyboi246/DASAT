pipeline {
    agent any

    tools {
        nodejs '23.11.0'
    }

    environment {
        target_name = "http://172.17.0.1:3000"
    }

    stages {
        stage('Clone') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    extensions: [],
                    userRemoteConfigs: [[
                        credentialsId: 'user-github',
                        url: 'https://github.com/markyboi246/DASAT'
                    ]]
                )
            }
        }

        stage('Build & Serve') {
            steps {
                dir('frontend') {
                    sh '''
                        echo "Installing dependencies (legacy peer deps)"
                        npm install --legacy-peer-deps

                        echo "Starting app in background"
                        nohup npm start > app.log 2>&1 &

                        echo "Waiting for app to be ready"
                        sleep 20
                    '''
                }
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    echo "Running ZAP Scan"

                    sh """
                        docker run --rm -t --network=host \
                        ghcr.io/zaproxy/zaproxy:stable zap-full-scan.py \
                        -t "${env.target_name}" \
                        -r zap_report.html || echo "ZAP Scan failed"
                    """

                    echo "Checking for ZAP report..."

                    archiveArtifacts artifacts: 'zap_report.html', fingerprint: true

                    if (fileExists('zap_report.html')) {
                        echo "ZAP Scan completed successfully, report generated."
                    } else {
                        echo "ZAP Scan failed to generate report."
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
    }
}

