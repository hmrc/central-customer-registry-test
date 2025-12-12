pipeline {
    agent {
        label "${env.EXT_PIPELINE_AGENT}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                sh '#!/bin/bash\n ls -l /home/jenkins/.local/bin'
                sh '#!/bin/bash\n ls -lR /opt/mvn'
                sh '#!/bin/bash\n mvn --version'
            }
        }
        stage('Build') {
            steps {
                echo "Building Fun Service..."
            }
        }
        stage('Test') {
            steps {
                echo "Running tests..."
            }
        }
        stage('Deploy') {
            steps {
                echo "Deploying application..."
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
