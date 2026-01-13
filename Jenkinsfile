pipeline {
    agent {
        label "${env.EXT_PIPELINE_AGENT}"
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                sh '/opt/apache-maven-3.9.11/bin/mvn --version'
            }
        }
        stage('Build') {
            steps {
                echo "Building Fun Service..."
                echo "$PIPELINE_BRANCH"
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
