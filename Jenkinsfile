pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE', defaultValue: 'b1ueroom/teedy-app', description: 'Docker Hub image name, for example yourname/teedy-app')
        string(name: 'DOCKER_HUB_CREDENTIALS', defaultValue: 'dockerhub_credentials', description: 'Jenkins credential ID for Docker Hub')
        string(name: 'CONTAINER_PREFIX', defaultValue: 'teedy-assignment', description: 'Prefix for the three running containers')
        string(name: 'HOST_PORT_1', defaultValue: '8082', description: 'Host port for container 1')
        string(name: 'HOST_PORT_2', defaultValue: '8083', description: 'Host port for container 2')
        string(name: 'HOST_PORT_3', defaultValue: '8084', description: 'Host port for container 3')
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        // Keep the runtime values in environment variables so shell steps can reuse them.
        DOCKER_HUB_CREDENTIALS = "${params.DOCKER_HUB_CREDENTIALS}"
        DOCKER_IMAGE = "${params.DOCKER_IMAGE}"
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        CONTAINER_PREFIX = "${params.CONTAINER_PREFIX}"
        HOST_PORT_1 = "${params.HOST_PORT_1}"
        HOST_PORT_2 = "${params.HOST_PORT_2}"
        HOST_PORT_3 = "${params.HOST_PORT_3}"
    }

    stages {
        stage('Build Maven Artifact') {
            steps {
                sh 'mvn -B -Pprod clean install -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_HUB_CREDENTIALS) {
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push()
                        docker.image("${DOCKER_IMAGE}:${DOCKER_TAG}").push('latest')
                    }
                }
            }
        }

        stage('Run Three Containers') {
            steps {
                sh '''
                    set -e

                    docker stop ${CONTAINER_PREFIX}-1 ${CONTAINER_PREFIX}-2 ${CONTAINER_PREFIX}-3 || true
                    docker rm ${CONTAINER_PREFIX}-1 ${CONTAINER_PREFIX}-2 ${CONTAINER_PREFIX}-3 || true

                    docker run -d \
                      --name ${CONTAINER_PREFIX}-1 \
                      -p ${HOST_PORT_1}:8080 \
                      -e DOCS_BASE_URL=http://localhost:${HOST_PORT_1} \
                      -v ${CONTAINER_PREFIX}-data-1:/data \
                      ${DOCKER_IMAGE}:${DOCKER_TAG}

                    docker run -d \
                      --name ${CONTAINER_PREFIX}-2 \
                      -p ${HOST_PORT_2}:8080 \
                      -e DOCS_BASE_URL=http://localhost:${HOST_PORT_2} \
                      -v ${CONTAINER_PREFIX}-data-2:/data \
                      ${DOCKER_IMAGE}:${DOCKER_TAG}

                    docker run -d \
                      --name ${CONTAINER_PREFIX}-3 \
                      -p ${HOST_PORT_3}:8080 \
                      -e DOCS_BASE_URL=http://localhost:${HOST_PORT_3} \
                      -v ${CONTAINER_PREFIX}-data-3:/data \
                      ${DOCKER_IMAGE}:${DOCKER_TAG}

                    docker ps --filter "name=${CONTAINER_PREFIX}"
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/**/*.war', fingerprint: true
        }
    }
}
