pipeline {
    agent {
        docker {
            image 'binarysearch/node-chrome-headless-alpine:1.0.1'
        }
    }
    environment {
        CI = 'true'
        DOCKER_USER = 'binarysearch'
    }
    stages {
        stage('Build') {
            steps {
                sh 'printenv'
                sh 'npm install && npm run ng build --prod'
            }
        }
        stage('Test') {
            steps {
                sh 'npm run test:app'
            }
        }
        stage('Deliver dev') {
            when {
                expression {
                    return env.BRANCH_NAME == 'develop'
                } 
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-password', variable: 'DOCKER_PASS')]) {
                        sh 'docker login --username=${DOCKER_USER} --password=${DOCKER_PASS}'
                    }
                    sh 'docker build --rm --build-arg app_version_arg=dev -f Dockerfile -t binarysearch/piros:dev .'
                    sh 'docker push binarysearch/piros:dev'
                    sh 'docker container rm piros-dev -f || true'
                    sh 'docker run -d -e API_HOST=https://api-dev.piros.org --network=dev_enviroment_default --network-alias=piros-dev --name=piros-dev binarysearch/piros:dev'
                }
            }
        }
        stage('Deliver') {
            when {
                expression {
                    return env.BRANCH_NAME == env.TAG_NAME
                } 
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker-password', variable: 'DOCKER_PASS')]) {
                        sh 'docker login --username=${DOCKER_USER} --password=${DOCKER_PASS}'
                    }
                    sh 'docker build --rm --build-arg app_version_arg=${TAG_NAME} -f Dockerfile -t binarysearch/piros:${TAG_NAME} .'
                    sh 'docker push binarysearch/piros:${TAG_NAME}'
                    sh 'docker container rm piros -f || true'
                    sh 'docker run -d -e API_HOST=https://api.piros.org --network=dev_enviroment_default --network-alias=piros --name=piros binarysearch/piros:${TAG_NAME}'
                }
            }
        }
    }
}
