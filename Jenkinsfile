pipeline {
    agent any

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Compile, Audit, and Test Application') {
            steps {
                echo '=== EXECUTING ENFORCED TESTING ENGINE ==='
                // 🚨 INTENTIONAL FAILURE INJECTION: Forces a non-zero exit code to simulate a failed unit test suite
                sh "exit 1"
            }
        }

        stage('Package Artifact') {
            steps {
                echo 'This stage will be skipped upon test failure.'
            }
        }
        
        stage('Deploy to Dev Environment') {
            steps {
                // 🚨 CRITICAL FIX: Changed propagate to true so downstream issues fail the parent build visibly
                build job: 'ansible-webserver-deployment', propagate: true, wait: true
            }
        }
    }
}
