// @Library('SharedLib') _

// pipeline {
//       agent any
//       tools {
//         maven 'Maven'
//        }

//     environment {
//         DOCKER_IMAGE = "hagert/multi-app"
//         DOCKER_BRANCH_NAME = "${BRANCH_NAME}"
//         DOCKERHUB_CRED_ID = "hagert"
//         NAMESPACE = "${BRANCH_NAME}"
//     }

//     stages {
//         stage('Checkout Code') {
//             steps {
//                 checkout scm
//             }
//         }

//         stage('Check Docker Access') {
//             steps {
//                 script {
//                     sh 'docker --version'
//                     sh 'docker ps || echo "Docker is not running"' // Check if Docker is running
//                 }
//             }
//         }
//         stage('Run Unit Tests') {
//             steps {
//                 runUnitTests()
                

//             }
//         }

//         stage('Build Application') {
//             steps {
//                 buildApp()
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     buildImage2("${DOCKER_IMAGE}", "${DOCKER_BRANCH_NAME}")
//                 }
//             }
//         }

//         stage('Push Docker Image') {
//             steps {
//                 script {
//                     pushImage2("${DOCKER_IMAGE}", "${DOCKER_BRANCH_NAME}", "${DOCKERHUB_CRED_ID}")
//                 }
//             }
//         }

//         stage('Remove Image Locally') {
//             steps {
//                 script {
//                     removeImageLocally2("${DOCKER_IMAGE}", "${DOCKER_BRANCH_NAME}")
//                 }
//             }
//         }

//         stage('Deploy to Kubernetes') {
//             steps {
//                 script {
//                     sh 'test -f k8s-deployment.yaml || (echo "k8s-deployment.yaml not found" && exit 1)'

//                     def KUBE_TOKEN_CRED_ID
//                     if (BRANCH_NAME == "main") {
//                         KUBE_TOKEN_CRED_ID = "main-namespace-token"
//                     } else if (BRANCH_NAME == "stag") {
//                         KUBE_TOKEN_CRED_ID = "stag-namespace-token"
//                     } else if (BRANCH_NAME == "dev") {
//                         KUBE_TOKEN_CRED_ID = "dev-namespace-token"
//                     } else {
//                         error("Unsupported branch: ${BRANCH_NAME}")
//                     }

//                     deployToK8s(NAMESPACE, KUBE_TOKEN_CRED_ID)
//                 }
//             }
//         }
//     }

//     post {
//         success {
//             echo "Pipeline executed successfully!"
//         }
//         failure {
//             echo "Pipeline failed."
//         }
//     }
// }
@Library('SharedLib') _

pipeline {
    agent any 
       tools {
         maven 'Maven'
        }
    environment {
        DOCKER_IMAGE = "hagert/multi-app"
        DOCKER_BRANCH_NAME = "${BRANCH_NAME}"
        DOCKERHUB_CRED_ID = "hagert"
        NAMESPACE = "${BRANCH_NAME}" // Automatically set namespace based on branch
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm // Automatically clone the repository based on the branch
            }
        }

        stage('Check Docker Access') {
            steps {
                script {
                    sh 'docker --version'
                    sh 'docker ps || echo "Docker is not running"' // Check if Docker is running
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                runUnitTests() // Use shared library function
            }
        }

        stage('Build Application') {
            steps {
                buildApp() // Use shared library function
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    buildImage2("${DOCKER_IMAGE}", "${DOCKER_BRANCH_NAME}") // Use shared library function
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    pushImage2("${DOCKER_IMAGE}", "${DOCKER_BRANCH_NAME}", "${DOCKERHUB_CRED_ID}") // Use shared library function
                }
            }
        }

        stage('Remove Image Locally') {
            steps {
                script {
                    removeImageLocally2("${DOCKER_IMAGE}", "${DOCKER_BRANCH_NAME}") // Use shared library function
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh 'test -f k8s-deployment.yaml || (echo "k8s-deployment.yaml not found" && exit 1)'

                    // Replace placeholders in k8s-deployment.yaml
                    sh """
                        sed -i.bak "s|{{ NAMESPACE }}|${NAMESPACE}|g" k8s-deployment.yaml
                        sed -i.bak "s|{{ DOCKER_IMAGE }}|${DOCKER_IMAGE}|g" k8s-deployment.yaml
                        sed -i.bak "s|{{ DOCKER_BRANCH_NAME }}|${DOCKER_BRANCH_NAME}|g" k8s-deployment.yaml
                    """

                    // Debugging: Check the updated k8s-deployment.yaml
                    sh "cat k8s-deployment.yaml"

                    // Deploy to Kubernetes
                    withCredentials([string(credentialsId: "${KUBE_TOKEN_CRED_ID}", variable: 'KUBE_TOKEN')]) {
                        sh """
                            kubectl config set-credentials jenkins-user --token=${KUBE_TOKEN}
                            kubectl config set-context --current --user=jenkins-user --namespace=${NAMESPACE}
                            kubectl config use-context --current
                            kubectl apply -f k8s-deployment.yaml
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline executed successfully!"
        }
        failure {
            echo "Pipeline failed."
        }
    }
}