pipeline {
    agent {
        label "${env.EXT_PIPELINE_AGENT}"
    }

    stages {
        stage('Checkout') {
            steps {
                sh 'echo ${AGENT_IMAGE_VERSION}'
                sh 'sfdx --help'
                sh 'which java'
                echo "Checking out source code..."
                sh 'mvn --version'
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
