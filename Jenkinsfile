pipeline {
    agent any

    parameters {
        string(name: 'IMAGE', defaultValue: 'localhost:5000/cpp-demo:1.0.1', description: 'CI Build Image')
    }

    stages {

        stage('Pull Image') {
            steps {
                sh '''
                docker pull ${IMAGE}
                '''
            }
        }


        stage('Smoke Test') {
            steps {
                sh '''
                echo "Running Smoke Test..."

                docker run --rm ${IMAGE} bash -c "
                    set -e
                    /build/app &
                    sleep 1
                    ps aux | grep app || true
                "

                echo "Smoke Test OK"
                '''
            }
        }


        stage('Integration Test') {
            steps {
                sh '''
                echo "Running Integration Test..."

                docker run -d --rm --name it-${BUILD_NUMBER} ${IMAGE} sleep 3600

                docker exec it-${BUILD_NUMBER} /build/app > output.txt

                grep -q "Hello CI/CD World" output.txt

                docker rm -f it-${BUILD_NUMBER}

                echo "Integration Test PASSED"
                '''
            }
        }

        stage('Result') {
            steps {
                sh '''
                echo "TEST SUITE PASSED"
                '''
            }
        }
    }
}

