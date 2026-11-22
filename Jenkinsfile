
pipeline {
    agent any

    tools {
        nodejs 'node'
    }

    environment {
        MAIN_IMAGE = 'nodemain:v1.0'
        DEV_IMAGE  = 'nodedev:v1.0'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Building branch: ${env.BRANCH_NAME}"
            }
        }

        stage('Build') {
            steps {
                echo 'Installing NodeJS dependencies...'
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                echo 'Running application tests...'
                sh 'CI=true npm test -- --watchAll=false'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh 'docker build -t nodemain:v1.0 .'
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh 'docker build -t nodedev:v1.0 .'
                    } else {
                        error "Unsupported branch: ${env.BRANCH_NAME}"
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh '''
                            docker rm -f cicd-main || true
                            docker run -d \
                              --name cicd-main \
                              --expose 3000 \
                              -p 3000:3000 \
                              -e HOST=0.0.0.0 \
                              nodemain:v1.0
                        '''
                    } else if (env.BRANCH_NAME == 'dev') {
                        sh '''
                            docker rm -f cicd-dev || true
                            docker run -d \
                              --name cicd-dev \
                              --expose 3001 \
                              -p 3001:3000 \
                              -e HOST=0.0.0.0 \
                              nodedev:v1.0
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "CI/CD pipeline completed successfully for ${env.BRANCH_NAME}"
        }
        failure {
            echo "CI/CD pipeline failed for ${env.BRANCH_NAME}"
        }
    }
}
