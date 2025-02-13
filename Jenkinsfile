pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building application for branch: ${env.BRANCH_NAME}"
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'master'
                    branch 'dev'
                    branch 'stage'
                }
            }
            steps {
                script {
                    echo "Deploying application for branch: ${env.BRANCH_NAME}"
                }
            }
        }
    }
}
