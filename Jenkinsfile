pipeline {
    agent any

    tools {
        nodejs '23.11.0'
    }

    environment {
        target_name = "https://owasp-juice.shop"
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
                        ghcr.io/zaproxy/zaproxy:stable zap-baseline.py \
                        -t "${env.target_name}" \
                       
                    """

}

                    }
                }
            }
        }
    }
}

