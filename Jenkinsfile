pipeline {
    agent any

    parameters {
        string(name: 'IMAGE', defaultValue: '', description: 'Image to test (e.g. localhost:5000/cpp-demo:1.0.10)')
        string(name: 'VERSION', defaultValue: '', description: 'Version tag')
    }

    environment {
        CONTAINER_NAME = "it-${BUILD_NUMBER}"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh """
                    set -e
                    echo "Pulling image: ${params.IMAGE}"
                    docker pull ${params.IMAGE}
                """
            }
        }

        stage('Smoke Test') {
            steps {
                sh """
                    set -e
                    echo "Running Smoke Test..."

                    docker run --rm ${params.IMAGE} bash -c '
                        set -e
                        /build/app &
                        sleep 1
                        echo "Smoke Test OK"
                    '
                """
            }
        }

        stage('Integration Test') {
            steps {
                sh """
                    set -e
                    echo "Running Integration Test..."

                    docker run -d --rm --name ${CONTAINER_NAME} ${params.IMAGE} sleep 3600

                    sleep 2

                    echo "Step 1: execute binary"
                    docker exec ${CONTAINER_NAME} /build/app

                    echo "Step 2: validate output"
                    docker exec ${CONTAINER_NAME} /build/app | grep -q "Hello CI/CD World"

                    docker rm -f ${CONTAINER_NAME} || true

                    echo "Integration Test PASSED"
		    build job: 'pipeline_deploy',
  		    	 parameters: [
        			string(name: 'IMAGE', value: "${params.IMAGE}"),
       		 		string(name: 'VERSION', value: "${params.VERSION}")
    			]
                """
            }
        }
    }

    post {
        always {
            sh "docker rm -f ${CONTAINER_NAME} || true"
        }

        success {
            echo "TEST PIPELINE SUCCESS ✅"
        }

        failure {
            echo "TEST PIPELINE FAILED ❌"
        }
    }
}
