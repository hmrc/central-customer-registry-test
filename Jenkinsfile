pipeline {
    agent {
        label "${env.EXT_PIPELINE_AGENT}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                sh '#!/bin/bash\n set'
                sh '#!/bin/bash\n ln -s /opt/apache-maven-3.9.11/bin/mvn /home/jenkins/.local/bin/mvn'
                sh '#!/bin/bash\n ls -l /home/jenkins/.local/bin'
                sh '#!/bin/bash\n ls -lR /opt'
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
